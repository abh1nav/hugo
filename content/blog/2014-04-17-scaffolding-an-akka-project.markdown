+++
title = "Scaffolding an Akka project"
date = "2014-04-17 01:34:01 -0400"
tags = ["scala", "akka", "sbt", "programming"]
type = "post"
aliases = [
  "/blog/2014/04/17/scaffolding-an-akka-project/"
]
+++

Every time I've needed to start a new Akka project, I've had to go through the process of scaffolding a new project from scratch. So I finally got around to creating a skeleton project that includes the bare minimum dependencies along with a build file, plugins and configuration required to create a fat jar as well as the ability to run in place. <!--more-->You can find the akka-skeleton project [here](https://github.com/abh1nav/akka-skeleton).
  
To get going with akka-skeleton,

1. Modify the `organization, name & version` in `build.sbt`  
2. Rename the package heirarchy under `src/main/scala`
3. Ensure you have atleast one class that `extends App`
4. `sbt run`

### Included Dependencies
1. Akka Actor Module
2. Akka Agent Module
3. Google Guava
4. Joda Time (and `joda-convert` to make it work correctly with Scala)
5. JUnit, ScalaTest and Akka TestKit
6. Akka SLF4J Adapter
  
### File Structure

The project root looks like this:
  
```
project/
src/
build.sbt
```
  
#### project
The `project` folder contains all the files that help SBT build the project. 
  
```
project
|-Dependencies.scala
|-build.properties
|-plugins.sbt
```
  
* `build.properties` describes the SBT version used to build the project  
* `plugins.sbt` describes all the SBT plugins used to build the project - for example, the `assembly` plugin is used to create a fat jar of the project including all the dependencies  
* `Dependencies.scala` describes all the project dependencies - objects from here are imported by the `build.sbt` file when building the project  
  
The `src` folder contains the project source, test and resource files.

### Build, Run and Assembly
`sbt clean` to clean.  
`sbt compile` to build.  
`sbt run` to run.  
`sbt assembly` to create a fat jar.  
