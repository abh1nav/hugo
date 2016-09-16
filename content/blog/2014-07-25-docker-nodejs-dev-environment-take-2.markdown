+++
title = "Docker + NodeJS Dev Environment: Take 2 - Container Linking"
date = "2014-07-25 10:31:08 -0400"
tags = ["docker", "javascript", "programming", "nodejs", "devops", "redis"]
type = "post"
aliases = [
	"/blog/2014/07/25/docker-nodejs-dev-environment-take-2/"
]
+++

I've had lots of feedback on my previous [post](/2014-06-17-develop-a-nodejs-app-with-docker/) and I'd like to share some of it which I found especially helpful.

[Mathieu Larose](https://github.com/larose) sent over a much shorter one-liner that installs Node in the docker container. This means my original `Dockerfile` step<!--more-->

<pre>
RUN	\
	cd /opt && \
	wget http://nodejs.org/dist/v0.10.28/node-v0.10.28-linux-x64.tar.gz && \
	tar -xzf node-v0.10.28-linux-x64.tar.gz && \
	mv node-v0.10.28-linux-x64 node && \
	cd /usr/local/bin && \
	ln -s /opt/node/bin/* . && \
	rm -f /opt/node-v0.10.28-linux-x64.tar.gz
</pre>

turns into

<pre>
RUN \
	wget -O - http://nodejs.org/dist/v0.10.29/node-v0.10.29-linux-x64.tar.gz \
	| tar xzf - --strip-components=1 --exclude="README.md" --exclude="LICENSE" \
	--exclude="ChangeLog" -C "/usr/local"
</pre>

The second piece of feedback was via Twitter from [Darshan Shankar](http://twitter.com/DShankar)

{{% screenshot "/images/2014-07-25-Feedback-2.png" %}}

As I explained on Twitter and at the end of the previous post, having Redis and Node in the same container was meant only to demonstrate how Docker works to first-timers. It isn't recommended as a production setup.

### Unbundling Redis and Container Links

Since I've already agreed that having Redis and Node together is probably not the greatest idea, I'm going to take this opportunity to fix this mistake and demonstrate container linking at the same time.

The fundamental idea is to strive for single-purpose Docker containers and then compose them together to build more complex systems. This implies we need to rip out the Redis related bits from our old Dockerfile and run Redis in a Docker container by itself.

### Redis in a Docker container
Now that we've decided to unbundle Redis, let's run it in its own container. Fortunately, as is often the case with Docker, someone else has already done the hard work for us. Running Redis locally is as simple as:

<pre>
docker run -d --name="myredis" -p 6379:6379 dockerfile/redis
</pre>

You'll notice the extra `--name="myredis"` parameter. We'll use that in the next step to tell our app's container about this Redis container.

### Dockerfile update
The next step is to update our Dockerfile and exclude the Redis-related instructions.

<pre>
FROM dockerfile/ubuntu

MAINTAINER Abhinav Ajgaonkar <abhinav316@gmail.com>

# Install pre-reqs
RUN	\
	apt-get -y -qq install python

# Install Node
RUN \
	wget -O - http://nodejs.org/dist/v0.10.29/node-v0.10.29-linux-x64.tar.gz \
	| tar xzf - --strip-components=1 --exclude="README.md" --exclude="LICENSE" \
	--exclude="ChangeLog" -C "/usr/local"

# Set the working directory
WORKDIR	/src

CMD ["/bin/bash"]
</pre>

### Build, Link and Run
Our app image can now be built with:

<pre>
docker build -t sqldump/docker-dev:0.2 .
</pre>

Once the build has completed, we can launch a container using the image. 
The command for launching an instance of this image also needs to be modified slightly:

<pre>
docker run -i -t --rm                     \
		-p 3000:3000                        \
		-v `pwd`:/src                       \
		--name="myapp"                      \
		--link myredis:myredis              \
		-e REDIS_HOST="myredis"             \
		sqldump/docker-dev:0.2
</pre>

Here, I've used the `--link` option to tell the `myapp` container about the `redis` container. The `--link` option allows linked containers to communicate securely over the `docker0` interface. When we link the `myredis` container to the `myapp` container, `myapp` can then access services on `myredis` just by using the hostname and the hostname-to-IP resolution will be handled transparently by Docker.

I've also injected an environment variable called `REDIS_HOST` using the `-e` flag to tell our node app where to find the linked Redis.

Once the container is running, you can utilize the methods described in the previous post to install dependencies and get your server running.

I hope this provides a satisfactory demonstration of how linked containers can be used to compose single-purpose docker containers into a more complex working system.