+++
title = "Tracing Akka Projects with Atmos"
date = "2014-04-19 16:37:28 -0400"
tags = ["scala", "akka", "sbt", "programming"]
type = "post"
aliases = [
  "/blog/2014/04/19/tracing-akka-projects-with-atmos/"
]
+++

[Atmos](https://github.com/sbt/sbt-atmos) is an SBT plugin that allows you to trace Akka and Play projects to help identify and fix bottlenecks. 

### Installation
Add the following lines to your project:
  
```scala
addSbtPlugin("com.typesafe.sbt" % "sbt-atmos" % "0.3.2")
```

```scala
atmosSettings

atmosTestSettings
```

### Usage
```
sbt atmos:run
```
The Atmos UI comes up on `localhost:9900`.

### Screenshots
Atmos has a great overview screen that shows you vital system-wide statistics.

{{% screenshot "/images/2014-04-19-Atmos-Overview.png" %}}

It also allows you to drill down and dig into dispatcher-level as well as ActorSystem level stats.

{{% screenshot "/images/2014-04-19-Atmos-Dispatchers.png" %}}

{{% screenshot "/images/2014-04-19-Atmos-ActorSystems.png" %}}

And, best of all, it shows you Actor-level stats like message throughput, mailbox size over time as well as mean time spent in mailbox- which are extremely helpful when tracing bottlenecks.

{{% screenshot "/images/2014-04-19-Atmos-Actor.png" %}}

I've put up a working project with Atmos [here](https://github.com/sqldump/akka-atmos). It'll run for about 5 minutes so you can explore the Atmos UI and then terminate. If you want to give it a try:

```
git clone https://github.com/sqldump/akka-atmos.git
cd akka-atmos
sbt atmos:run
```