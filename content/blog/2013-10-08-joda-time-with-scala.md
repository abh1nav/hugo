+++
title = "Joda Time with Scala"
date = "2013-10-08T14:25:02-04:00"
tags = ["scala", "akka", "programming"]
type = "post"
aliases = [
    "/blog/2013/10/08/joda-time-with-scala/"
]
+++

When trying to use Joda-Time in a Scala project, I encountered a rather cryptic error:<!--more-->

<pre>
scala: error while loading Instant, class file '/Users/asdf/.m2/repository/joda-time/joda-time/2.3/joda-time-2.3.jar(org/joda/time/Instant.class)' is broken
(class java.lang.RuntimeException/bad constant pool tag 9 at byte 48)
</pre>
  
This can be solved by adding the [joda-convert](http://mvnrepository.com/artifact/org.joda/joda-convert/1.5) dependency into Maven/SBT.
