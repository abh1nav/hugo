+++
title = "Develop a NodeJS app with Docker"
date = "2014-06-17 01:58:40 -0400"
tags = ["docker", "javascript", "programming", "nodejs"]
type = "post"
aliases = [
    "/blog/2014/06/17/develop-a-nodejs-app-with-docker/"
]
+++

This is the first of two posts. This post covers a somewhat detailed tutorial on using Docker as a replacement for [Vagrant](http://www.vagrantup.com/) when developing a Node app using the [Express](http://expressjs.com/) framework. To make things a bit non-trivial, the app will persist session information in Redis using the [connect-redis](https://github.com/visionmedia/connect-redis) middleware. The second post will cover productionizing this development setup.
  
### The Node App
The app consists of a `package.json`, `server.js` and a `.gitignore` file, which is about as simple as it gets.

<pre>
node_modules/*
</pre>

```json
{
    "name": "docker-dev",
    "version": "0.1.0",
    "description": "Docker Dev",
    "dependencies": {
        "connect-redis": "~1.4.5",
        "express": "~3.3.3",
        "hiredis": "~0.1.15",
        "redis": "~0.8.4"
    }
}
```
  
```javascript
var express = require('express'),
    app = express(),
    redis = require('redis'),
    RedisStore = require('connect-redis')(express),
    server = require('http').createServer(app);

app.configure(function() {
  app.use(express.cookieParser('keyboard-cat'));
  app.use(express.session({
        store: new RedisStore({
            host: process.env.REDIS_HOST || 'localhost',
            port: process.env.REDIS_PORT || 6379,
            db: process.env.REDIS_DB || 0
        }),
        cookie: {
            expires: false,
            maxAge: 30 * 24 * 60 * 60 * 1000
        }
    }));
});

app.get('/', function(req, res) {
  res.json({
    status: "ok"
  });
});

var port = process.env.HTTP_PORT || 3000;
server.listen(port);
console.log('Listening on port ' + port);
```

`server.js` pulls in all the dependencies and starts an express app. The express app is configured to store session information in Redis and exposes a single endpoint that returns a status message as JSON. Pretty standard stuff.  
  
One thing to note here is that the connection information for redis can be overridden using environment variables - this will be useful later on when moving from dev to prod.  
  
### The Dockerfile
For development, we'll have redis and node running in the same container. To make this happen, we'll use a Dockerfile to configure the container.

<pre>
FROM dockerfile/ubuntu

MAINTAINER Abhinav Ajgaonkar <abhinav316@gmail.com>

# Install Redis
RUN	\
	apt-get -y -qq install python redis-server

# Install Node
RUN	\
	cd /opt && \
	wget http://nodejs.org/dist/v0.10.28/node-v0.10.28-linux-x64.tar.gz && \
	tar -xzf node-v0.10.28-linux-x64.tar.gz && \
	mv node-v0.10.28-linux-x64 node && \
	cd /usr/local/bin && \
	ln -s /opt/node/bin/* . && \
	rm -f /opt/node-v0.10.28-linux-x64.tar.gz

# Set the working directory
WORKDIR	/src

CMD ["/bin/bash"]
</pre>

Taking it line by line,

<pre>
FROM dockerfile/ubuntu
</pre>

This tells docker to use the `dockerfile/ubuntu` image provided by Docker Inc. as the base image for the build.

<pre>
RUN	\
	apt-get -y -qq install python redis-server
</pre>

The base image contains absolutely nothing- so we need to apt-get everything needed for our app to run. This statement installs python and redis-server. Redis server is required because we'll be storing session info in it and python is required by npm to be able to build the C-extension used by the redis node module.
  
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
  
This downloads and extracts the 64-bit NodeJS binaries.
  
<pre>
WORKDIR	/src
</pre>
  
This tells docker to `cd /src` once the container has started, before executing what's specified in the `CMD` property.
  
<pre>
CMD ["/bin/bash"]
</pre>
  
Launch `/bin/bash` as a final step.

### Build and run the container

Now that the docker file is written, let's build a Docker image.
  
<pre>
docker build -t sqldump/docker-dev:0.1 .
</pre>
  
Once the image done building, we can launch a container using:

<pre>
docker run -i -t --rm \
           -p 3000:3000 \
           -v `pwd`:/src \
           sqldump/docker-dev:0.1
</pre>
  
Let's see what's going on in the docker run command.
  
`-i` starts the container in interactive mode (versus -d for detached mode). This means the container will exit once the interactive sessions is over.
  
`-t` allocates a pseudo-tty.
  
`--rm` removes the container and its filesystem on exit.
  
`-p 3000:3000` forwards port 3000 on to the host to port 3000 on the container.
  
<pre>
-v `pwd`:/src
</pre>

This mounts the current working directory in the host (i.e. our project files) to `/src` inside the container. We mount the current folder as a volume rather than using the `ADD` command in the Dockerfile so that any changes we make to the files in a text editor will be seen by the container right away.

`sqldump/docker-dev:0.1` the name and version of the docker image to run - this is the same one we used when building the docker image.

Since the Dockerfile specifies `CMD ["/bin/bash"]`, we're dropped into a bash shell once the container has started. If the docker run command succeeds, it'll look something like this:

{{% screenshot "/images/2014-06-17-Docker-Run.png" %}}
  
### Start Developing

Now that the container is running, we'll need to get a few standard, non-docker related things sorted out before we can start writing code. First, start redis server inside the container using:

```
service redis-server start
```

Then, install project dependencies and `nodemon`. [Nodemon](https://github.com/remy/nodemon) watches for changes in project files and restarts the server as needed. 

```
npm install
npm install -g nodemon
```

Finally, start up the server using:

```
nodemon server.js
```

Now, if you go to `http://localhost:3000` in your browser, you should see something like this:

{{% screenshot "/images/2014-06-17-First-Run.png" %}}

Let's add another endpoint to `server.js` to simulate development workflow:

```javascript
app.get('/hello/:name', function(req, res) {
  res.json({
    hello: req.params.name
  });
});
```

You should see that nodemon has detected your changes and restarted the server:

{{% screenshot "/images/2014-06-17-Reload.png" %}}

And now, if you point your browser to `http://localhost:3000/hello/world`, you should see the response:

{{% screenshot "/images/2014-06-17-Second-Run.png" %}}

### Production

The container, in its current state, is nowehere near production-ready. The data in redis won't be persisted across container restarts, i.e. if you restart the container, you'll have effectively blown away all your session data. The same thing will happen if you destroy the container and start a new one. Presumably, this is not what you want. I'll cover production setup in part II.
