+++
title = "Getting the Titan-0.3.2 Java Driver to work in Scala"
date = "2013-12-10T14:32:38-05:00"
tags = ["scala", "titan", "graphdb", "cassandra"]
type = "post"
aliases = [
    "/blog/2013/12/10/getting-the-titan-0-3-2-java-driver-to-work-in-scala/"
]
+++

Attempting to use the Java driver for Titan-0.3.2 (the current stable Titan release) with Cassandra v1.2.6 in a Scala project throws the following Astyanax error:<!--more-->  
```scala
java.lang.NoSuchMethodError: 
org.apache.cassandra.thrift.TBinaryProtocol: 
method (Lorg/apache/thrift/transport/TTransport;)V not found
```
This can be traced back to a [bug](https://github.com/Netflix/astyanax/issues/352) in Astyanax v1.56.37 which was fixed in v1.56.43.
This can be fixed by ensuring that [this](http://mvnrepository.com/artifact/com.netflix.astyanax/astyanax/1.56.43) dependency is listed above the Titan driver in your pom.xml or sbt.build.
