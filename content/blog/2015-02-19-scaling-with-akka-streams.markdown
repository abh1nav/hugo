+++
title = "Scaling with Akka Streams: Fetch 1M Twitter Profiles in less than 5min"
date = "2015-02-19 21:15:09 -0500"
tags = ["scala", "akka", "programming", "reactive", "streams", "twitter"]
type = "post"
aliases = [
  "/blog/2015/02/19/scaling-with-akka-streams/"
]
+++

During these past few weeks, I've had a chance to play around quite a bit with the soon-to-be-1.0 [Akka Streams](http://doc.akka.io/docs/akka-stream-and-http-experimental/1.0-M3/scala.html) module and I've been looking for a concrete reason to write about why this is an important milestone for the stream processing ecosystem. It just so happens that this week, we were conducting an experiment related to Twitter follower analysis that required pulling a few million public user profiles of followers of large Twitter accounts and the ability to do so in a relatively short amount of time. I decided to use Akka Streams for this project to see how far I could push it in a real world scenario.

## App Skeleton

I decided to start with a regular Scala command line app and hardcode the Twitter ID of the person whose followers' profiles we were attempting to retrieve (since this is going to be throwaway code just for the sake of the experiment). The only dependencies were [akka-actor](http://mvnrepository.com/artifact/com.typesafe.akka/akka-actor_2.11/2.3.9), [akka-stream-experimental](http://mvnrepository.com/artifact/com.typesafe.akka/akka-stream-experimental_2.11/1.0-M3) and [twitter4j-core](http://mvnrepository.com/artifact/org.twitter4j/twitter4j-core/4.0.2).


### Twitter API Interaction

We need some way to call the Twitter API primarily for two reasons:

1) Fetch all follower IDs of a given userID  
2) Fetch a user profile given a userID.  

Any call made to Twiter's API needs an authenticated `twitter4j.Twitter` object. Given a set of credentials, a `twitter4j.Twitter` object is constructed as follows:

```scala
import twitter4j.{Twitter, TwitterFactory}
import twitter4j.auth.AccessToken
import twitter4j.conf.ConfigurationBuilder

object TwitterClient {
    // Fill these in with your own credentials
    val appKey: String = ""
    val appSecret: String = ""
    val accessToken: String = ""
    val accessTokenSecret: String = ""

    def apply(): Twitter = {
        val factory = new TwitterFactory(new ConfigurationBuilder().build())
        val t = factory.getInstance()
        t.setOAuthConsumer(appKey, appSecret)
        t.setOAuthAccessToken(new AccessToken(accessToken, accessTokenSecret))  
        t
    }    
}
```

#### Given a user, return all followers

Using the `TwitterClient`, we can fetch a user's profile and list of followers with the following `TwitterHelpers` object:

```scala
object TwitterHelpers {
    // Lookup user profiles in batches of 100
    def lookupUsers(ids: List[Long]): List[User] = {
      val client = TwitterClient()
      val res = client.lookupUsers(ids.toArray)
      res.toList
    }

    // Fetch the IDs of a user's followers in batches of 5000
    def getFollowers(userId: Long): Try[Set[Long]] = {
      Try({
        val followerIds = mutable.Set[Long]()
        var cursor = -1L
        do {
          val client = TwitterClient()
          val res = client.friendsFollowers().getFollowersIDs(userId, cursor, 5000)
          res.getIDs.toList.foreach(x => followerIds.add(x))
          if (res.hasNext) {
            cursor = res.getNextCursor
          }
          else {
            cursor = -1 // Exit the loop
          }
        } while (cursor > 0)
        followerIds.toSet
      })
    }
}
```

### Akka Boilerplate

With the Twitter interactions out of the way, we can create a new command line app using:

```scala
object Main extends App {

  // ActorSystem & thread pools
  implicit val system: ActorSystem = ActorSystem("centaur")
  val executorService: ExecutorService = Executors.newCachedThreadPool()
  val ec: ExecutionContext = ExecutionContext.fromExecutorService(executorService)
  val log: LoggingAdapter = Logging.getLogger(system, Main)
  implicit val materializer = ActorFlowMaterializer()(system)

  // Put Stream code here
  //
  //
}
```

At this point, we have a working app that can be invoked using `sbt run`. Let's get started on the Akka Streams bit.

### The Pipeline

```scala
val startTime = System.nanoTime()
val userId = 410939902L // @headinthebox ~12K followers
val output = new ConcurrentLinkedQueue[String]()
Console.println(s"Fetching follower profiles for $userId")
Source(() => TwitterHelpers.getFollowers(userId).get.toIterable.iterator)
  .grouped(100)
  .map(x => TwitterHelpers.lookupUsers(x.toList))
  .mapConcat(identity)
  .runForeach(x => output.offer(x.getScreenName))
  .onComplete({
    case _ =>
      Console.println(s"Fetched ${output.size()} profiles")
      val endTime = System.nanoTime()
      Console.println(s"Time taken: ${(endTime - startTime)/1000000000.00}s")
      system.shutdown()
      Runtime.getRuntime.exit(0)
  }) (ec)
```

Breaking down the flow line by line
```scala
Source(() => TwitterHelpers.getFollowers(userId).get.toIterable.iterator)
```
turns the `Set[Long]` of follower IDs into an Akka Streams `Source`.
```scala
.grouped(100) // outputs a stream of Seq[Long]
.map(x => TwitterHelpers.lookupUsers(x.toList)) // outputs a stream of List[User]
.mapConcat(identity) // flattens the stream of List[User] into a stream of User
```
The `group`, `map` and `mapConcat` transform the stream of Longs into a stream of `twitter4j.User` objects.
```scala
.runForeach(x => output.offer(x.getScreenName))
```
And finally, the user objects are piped into a `Sink` which adds them to our output queue.

### Run 1 - 12,199 profiles in 108s

Running this flow takes 108 seconds to retrieve 12,199 user profiles from Twitter.

{{% screenshot "/images/2015-02-19-Attempt-1.png" %}}

### Run 2 - 12,200 profiles in 11s

Modifying the flow slightly to allow for more concurrency helps bring down the total time taken by a large value. The obvious bottleneck in the flow implementation is the synchronous fetching of user profiles in the stage where User IDs are mapped to User profile objects. Replacing the
```scala
.map(x => TwitterHelpers.lookupUsers(x.toList))
```
with
```scala
.mapAsyncUnordered[List[User]](x => Future[List[User]]({ TwitterHelpers.lookupUsers(x.toList) } (ec)))
```
reduces the time taken from **108**s to **11**s. **That's almost 10x faster with a single line change!**

{{% screenshot "/images/2015-02-19-Attempt-2.png" %}}

(It looks like Erik Meijer has gained a follower between our two runs).

### Run 3 - 1.22M profiles in 256s
[Netflix USA](https://twitter.com/netflix) has approximately 1.22M followers. Fetching followers for this account took 256s.

{{% screenshot "/images/2015-02-19-Attempt-3.png" %}}

### Run 4 - 2.88M profiles in 647s

Twitter co-founder [Jack Dorsey](https://twitter.com/jack) has 2.88M followers and the pipeline processed them in 647 seconds.

{{% screenshot "/images/2015-02-19-Attempt-4.png" %}}

Since Netflix (1.22M) was processed in 256s and Jack (2.88M) was processed in ~650s, it doesn't look like the pipeline is showing any signs of exhaustion as larger accounts are being processed.

### Final Thoughts

Before Akka Streams, creating a flow like this would require hand coding each stage as an actor, manually wiring everything up and carefully managing backpressure using a hybrid push/pull system alongwith finely configured timeouts and inbox sizes. Having worked on many such systems at [CrowdRiff](http://crowdriff.com), my experience so far has been mostly positive. Akka delivers exactly what it promises - an easy way to think about concurrency, excellent performance and a great toolkit to build distributed systems. However, once built, these systems often tend to be very complex and common changes like adding stages to the pipeline or modifying existing stages have to be done very carefully (despite unit tests!). Akka Streams takes this to a whole new level by allowing the user to create arbitrary flows (check out [stream graphs](http://doc.akka.io/docs/akka-stream-and-http-experimental/1.0-M3/scala/stream-graphs.html)) in simple and easy to read manner AND managing the backpressure for you! The large quantity of awesome in this module is certainly appreciated by me and my team - many thanks to the Akka folks and everyone who contributed to the Reactive Streams project. Happy hAkking!

