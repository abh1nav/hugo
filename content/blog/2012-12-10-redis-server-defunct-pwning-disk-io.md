+++
title = "Redis Server pwning your disk IO?"
date = "2012-12-10T13:11:58-05:00"
tags = ["devops", "redis"]
type = "post"
aliases = [
    "/blog/2012/12/10/redis-server-defunct-pwning-disk-io/"
]
+++
Run as root:<!--more-->
```
echo 1 > /proc/sys/vm/overcommit_memory
```
And restart your redis-server process.
