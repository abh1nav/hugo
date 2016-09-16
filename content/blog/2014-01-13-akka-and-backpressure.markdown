+++
title = "Akka and Back-pressure"
date = "2014-01-13 12:46:04 -0500"
tags = ["scala", "akka", "programming"]
type = "post"
aliases = [
    "/blog/2014/01/13/akka-and-backpressure/"
]
+++

Two steps are needed in order to correctly apply back-pressure in an Akka system:  <!--more-->

#### Step 1: Bounded Mailboxes and Push Timeouts
The default mailbox for an actor is an `UnboundedMailbox` backed by Java's `ConcurrentLinkedQueue`. As the name indicates, this mailbox grows without bound and will end up crashing the JVM with an `OutOfMemoryError` if the consumer significantly slower than the producer. If we want to be able to signal the producer to slow down, the first step is to switch to a `BoundedMailbox` backed by Java's `LinkedBlockingQueue` that will block the producer if the mailbox is full. More info about different types of mailboxes can be found [here](http://doc.akka.io/docs/akka/snapshot/scala/mailboxes.html).  
  
Blocking the producer forever is not a good solution because: `Rule #1 of Akka => don't block inside actors`. The solution to this problem is provided to us by Akka in the form of a `push timeout` for an Actor's mailbox. A push timeout is exactly what it sounds like: when you try to push a message to an actor's mailbox, if the mailbox is full, the action will timeout and the message will get routed to the `DeadLetterActorRef`.  
  
Configuring an actor to use a bounded mailbox with a 1000 message capacity and a push timeout of 100ms requires the following addition to the `application.conf`:  

```
bounded-mailbox {
	mailbox-type = "akka.dispatch.BoundedMailbox"
	mailbox-capacity = 1000
	mailbox-push-timeout-time = 100ms
}
```

The actor can then be initialized as follows

```scala
val actor = system.actorOf(Props[MyActorClass].withMailbox("bounded-mailbox"))
```

#### Step 2: DeadLetter Watcher
When an actor's mailbox is full and sent messages start timing out, they get routed to the `DeadLetterActorRef` via the [Event Stream](http://doc.akka.io/docs/akka/2.2.3/scala/event-bus.html#event-stream-scala) of the actor system. Akka allows actors to subscribe to event streams and listen in on all, or a filtered subset of, the messages flying around in the actor system. Since the dead letters service also utilizes the event stream infrastructure, we can subscribe to all `DeadLetter` messages being published in the stream and signal the producer to slow down.  
  
The following snipped can be used to get an actor subscribed to all the `DeadLetter` messages in a system

```scala
system.eventStream.subscribe(listeningActor, classOf[DeadLetter])
```

### Tying it all together with an Example
In this example, a fast producer sends messages to a slow consumer. The slow consumer has a bounded mailbox of size 10 and a push timeout of 10ms.  

##### SlowReceiver
The `SlowReceiver` blocks for 100ms after printing each message it receives. The blocking is only present to ensure that it's mailbox fills up.


```scala
import akka.actor._

class SlowReceiver extends Actor with ActorLogging {
  override def postStop() {
    log.info("SlowReceiver#postStop")
  }

  def receive: Actor.Receive = {
    case msg: String =>
      log.info(s"Received: $msg")
      Thread.sleep(100)
  }
}
```

##### FastSender
The `FastSender` waits for a kickoff message and then sends 15 messages to the `SlowReceiver` and a `PoisonPill` to itself. After terminating itself, the actor's `postStop` hook schedules a `PoisonPill` to be sent to the `SlowReceiver` 3 seconds after the `FastSender` has been terminated.


```scala
import akka.actor._
import scala.concurrent.duration._

class FastSender(slow: ActorRef) extends Actor with ActorLogging {
  override def postStop() {
    log.info("FastSender#postStop")
    context.system.scheduler.scheduleOnce(2.seconds, slow, PoisonPill)(context.dispatcher)
  }

  def receive: Actor.Receive = {
    case _ =>
      for(i <- 1 to 15) {
        slow ! s"[$i]"
      }
      log.info("Done sending all")
      self ! PoisonPill
  }
}
```

##### Watcher
The `Watcher` watches for and prints `DeadLetter`s being sent to the `SlowReceiver`. It also `context.watch`es the `SlowReceiver` and terminates the actor system when the `SlowReceiver` is killed.


```scala
import akka.actor._

class Watcher(target: ActorRef) extends Actor with ActorLogging {
  private val targetPath = target.path

  override def preStart() {
    context.watch(target)
  }

  def receive: Actor.Receive = {
    case d: DeadLetter =>
      if(d.recipient.path.equals(targetPath)) {
        log.info(s"Timed out message: ${d.message.toString}")
      }

    case Terminated(`target`) =>
      context.system.shutdown()
  }
}
```

##### BackPressureTest App
The App that ties it all together.

```scala
object BackPressureTest extends App {
  case object Ping

  val sys = ActorSystem("testSys")
  val slow = sys.actorOf(Props[SlowReceiver].withMailbox("bounded-mailbox"), "slow")
  val fast = sys.actorOf(Props(classOf[FastSender], slow), "fast")
  val watcher = sys.actorOf(Props(classOf[Watcher], slow), "watcher")
  sys.eventStream.subscribe(watcher, classOf[DeadLetter])

  fast ! Ping
}
```

##### Output
Running the `BackPressureTest` app gives the following output:
```
[INFO] [01/13/2014 14:00:56.303] [akka://testSys/user/slow] Received: [1]
[INFO] [01/13/2014 14:00:56.315] [akka://testSys/user/watcher] Timed out message: [12]
[INFO] [01/13/2014 14:00:56.326] [akka://testSys/user/watcher] Timed out message: [13]
[INFO] [01/13/2014 14:00:56.337] [akka://testSys/user/watcher] Timed out message: [14]
[INFO] [01/13/2014 14:00:56.347] [akka://testSys/user/fast] Done sending all
[INFO] [01/13/2014 14:00:56.347] [akka://testSys/user/watcher] Timed out message: [15]
[INFO] [01/13/2014 14:00:56.350] [akka://testSys/user/fast] FastSender#postStop
[INFO] [01/13/2014 14:00:56.403] [akka://testSys/user/slow] Received: [2]
[INFO] [01/13/2014 14:00:56.504] [akka://testSys/user/slow] Received: [3]
[INFO] [01/13/2014 14:00:56.605] [akka://testSys/user/slow] Received: [4]
[INFO] [01/13/2014 14:00:56.705] [akka://testSys/user/slow] Received: [5]
[INFO] [01/13/2014 14:00:56.807] [akka://testSys/user/slow] Received: [6]
[INFO] [01/13/2014 14:00:56.907] [akka://testSys/user/slow] Received: [7]
[INFO] [01/13/2014 14:00:57.008] [akka://testSys/user/slow] Received: [8]
[INFO] [01/13/2014 14:00:57.109] [akka://testSys/user/slow] Received: [9]
[INFO] [01/13/2014 14:00:57.209] [akka://testSys/user/slow] Received: [10]
[INFO] [01/13/2014 14:00:57.310] [akka://testSys/user/slow] Received: [11]
[INFO] [01/13/2014 14:00:58.367] [akka://testSys/user/slow] SlowReceiver#postStop
[DEBUG] [01/13/2014 14:00:58.373] [EventStream] shutting down: StandardOutLogger started
[DEBUG] [01/13/2014 14:00:58.373] [EventStream] shutting down: StandardOutLogger started
[DEBUG] [01/13/2014 14:00:58.375] [EventStream] all default loggers stopped

Process finished with exit code 0
```

##### Back-pressure Strategy
While this example doesn't actually implement back-pressure, it provides the infrastructure for applying a back-pressure strategy. A possible strategy would be to send `FastSender` a `SlowDown` message from within the `Watcher` for each dead letter received. The `SlowDown` case class could be defined as


```scala
case class SlowDown(dl: DeadLetter)
```

When `FastSender` receives a `SlowDown` message, it could either throttle itself or tell its upstream systems to slow down. The `SlowDown` message also encapsulates the relevant `DeadLetter` object to allow for retry logic.