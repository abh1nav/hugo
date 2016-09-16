+++
title = "Unit Testing Futures with ScalaTest"
date = "2014-09-08 17:08:08 -0400"
tags = ["scala", "testing", "futures"]
type = "post"
+++

Unit testing methods that return futures in Scala is quite straightforward using ScalaTest 2.0+<!--more-->

```scala
import org.scalatest.FunSuite
import org.scalatest.concurrent.ScalaFutures
import scala.concurrent.Future

class NiceClass$Test extends FunSuite with ScalaFutures {
  test("Test a method that returns a future") {
    val f: Future[Boolean] = somethingThatReturnsAFuture()

    whenReady(f) { result =>
      assert(result)
    }
  }
}
```
  
If your future is making a web service call or doing some sort of IO that takes a few seconds to complete, you might encounter an error like this one:

<blockquote>
<code>
A timeout occurred waiting for a future to complete. 
Queried 11 times, sleeping 15 milliseconds between each query.
</code>
</blockquote>

You can fix this by telling ScalaFutures how long it should to wait before declaring that the future has timed out.
  
```scala
import org.scalatest.time.{Millis, Seconds, Span}

implicit val defaultPatience = 
  PatienceConfig(timeout = Span(5, Seconds), interval = Span(500, Millis))
```
  
Happy testing!