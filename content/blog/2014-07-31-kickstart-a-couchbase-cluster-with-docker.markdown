+++
title = "Kickstart a Couchbase cluster with Docker"
date = "2014-07-31 11:01:15 -0400"
tags = ["docker", "python", "programming", "couchbase"]
type = "post"
aliases = [
    "/blog/2014/07/31/kickstart-a-couchbase-cluster-with-docker/"
]
+++

A few days ago, I was officially introduced to [Couchbase](http://www.couchbase.com/) at a [Toronto Hadoop User Group meetup](http://www.meetup.com/TorontoHUG/events/191410172/). I say "officially" because I've known about some Couchbase use-cases / pros and cons on a high level since v1.8, but never really had the time to look at it in detail. 

As the presentation progressed, it got me interested in actually tinkering around with a Couchbase cluster (kudos to [Don Pinto](https://twitter.com/NoSQLDon)). My first instinct was to head over to the [Docker Registry](https://registry.hub.docker.com) and do a quick search for `couchbase`. Using the `dustin/couchbase` image, I was able to get a 5-node cluster running in under 5 minutes.

### Run 5 containers

```bash
docker run -d -p 11210:11210 -p 11211:11211 -p 8091:8091 -p 8092:8092 --name cb1 dustin/couchbase
docker run -d --name cb2 dustin/couchbase
docker run -d --name cb3 dustin/couchbase
docker run -d --name cb4 dustin/couchbase
docker run -d --name cb5 dustin/couchbase
```

### Find Container IPs
Once the containers were up, I used `docker inspect` to find their internal IPs (usually in the `172.17.x.x` range). For example, `docker inspect cb1` returns

<pre><code lang="json">[{
    "Args": [
        "run"
    ],
    "Config": {
        "AttachStderr": false,
    ...
    "NetworkSettings": {
        "Bridge": "docker0",
        "Gateway": "172.17.42.1",
        "IPAddress": "172.17.0.27",
    ...
</code>
</pre>

Update: [Nathan LeClaire](https://twitter.com/upthecyberpunks) from [Docker](http://docker.io) was kind enough to write up a [quick script](https://gist.github.com/nathanleclaire/c7c402f7a9889ca77b98) that combines these two steps:

<pre><code lang="bash">docker run -d -p 11210:11210 -p 11211:11211 -p 8091:8091 -p 8092:8092 --name cb1 dustin/couchbase

for name in cb{2..5}; do 
    docker run -d --name $name dustin/couchbase
done

for name in cb{1..5}; do
    docker inspect -f '{{ .NetworkSettings.IPAddress }}' $name
done
</code>
</pre>

### Setup Cluster using WebUI
If `cb1` is at `172.17.0.27`, then the Couchbase management interface comes up at `http://172.17.0.27:8091` and the default credentials are:

<pre>
Username: Administrator
Password: password
</pre>

Once you're in, setting up a cluster is as easy as clicking "Add Server" and giving it the IPs of the other containers. As soon as you add a new server to the cluster, Couchbase will prompt you to run a "Cluster Rebalance" operation - hold off until you've added all 5 nodes and then run the rebalance.

{{% screenshot "/images/2014-07-31-Couchbase-WebUI-Rebalance.png" %}}

### Push some data into the cluster
Once the cluster was up, I wanted to get a feel for how the WebUI works so I wrote this script to grab some data from our existing cluster of JSON-store-that-I-am-too-ashamed-to-mention and added it to Couchbase:

```python
#!/usr/bin/env python
# encoding: utf-8

from pymongo import MongoClient
from couchbase import Couchbase

src = MongoClient(['m1', 'm2', 'm3'])['twitter_raw']['starbucks']
tar = Couchbase.connect(bucket='twitter_starbucks',  host='localhost')

for doc in src.find():
	key = str(doc['_id'])
	del doc['_id']
	result = tar.set(key, doc)

```

The Couchbase [python client](http://www.couchbase.com/communities/python) depends on [libcouchbase](http://www.couchbase.com/communities/c-client-library). Once those two were installed, and the `twitter_starbucks` bucket had been created in Couchbase, I was able to load ~100k JSON documents in a matter of minutes.