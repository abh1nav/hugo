+++
title = "ExecutionContext for Long Running Tasks"
date = "2014-01-08T15:20:32-05:00"
tags = ["scala", "akka", "programming"]
type = "post"
aliases = [
    "/blog/2014/01/08/executioncontext-for-long-running-tasks/"
]
+++

Rule #1 of Akka: don’t block inside actors. If you do have blocking / high latency calls, wrap them in a future and toss them into a different execution context specifically meant for high latency tasks.<!--more-->
```scala
import java.util.concurrent.Executors

class ClassyActor extends Actor {
  val numThreads = 10
  val pool = Executors.newFixedThreadPool(numThreads)
  val ctx = ExecutionContext.fromExecutorService(pool)
  
  def receive: Actor.Receive = {
    // An message that needs some high latency work done
    case m: Message =>
      val future = Future {
        // do something with m: Message
      } (ctx)

      future.onComplete({
        case Success(s) =>
          // do something when the task successfully completed
        case Failure(f) =>
          // do something when the task failed
      }) (ctx)
  }
}
```
