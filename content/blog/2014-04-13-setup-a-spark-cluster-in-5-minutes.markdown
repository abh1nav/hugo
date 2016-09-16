+++
title = "Setup a Spark Cluster in 5 minutes"
date = "2014-04-13 23:31:45 -0400"
tags = ["scala", "spark", "devops"]
type = "post"
aliases = [
  "/blog/2014/04/13/setup-a-spark-cluster-in-5-minutes/"
]
+++

#### Prerequisites
Assuming you have 3 nodes: `node1, node2, node3`, ensure the hosts file contains the following entries on all nodes:<!--more-->
  
```bash
192.168.1.100  node1
192.168.1.101  node2
192.168.1.102  node3
```
  
Be sure replace `192.168.x.x` with the actual IPs for each node.
  
#### Get Spark
Download binaries from http://spark.apache.org/downloads.html and extract it to `/opt/`.  

#### Spark Master
Start the master process on `node1`
  
```bash
cd /opt/spark-0.9.0-incubating-bin-hadoop1
sbin/start-master.sh
```

#### Spark Workers
Start worker processes on each node:
  
```bash
cd /opt/spark-0.9.0-incubating-bin-hadoop1
bin/spark-class org.apache.spark.deploy.worker.Worker spark://node1:7077
```
  
Check the Spark UI at `http://node1:8080` to make sure all workers have registered with the master.