+++
title = "MongoDB 2.4.0 EXT4 Readahead Warnings"
date = "2013-03-20T18:01:00-04:00"
tags = ["devops", "mongodb", "linux"]
type = "post"
aliases = [
    "/blog/2013/03/20/mongodb-2-4-0-ext4-readahead-warnings/"
]
+++

After deploying MongoDB 2.4.0 for the first time and connecting to it from a remote shell, it warned me:<!--more-->
```
MongoDB shell version: 2.4.0
connecting to: abc.com/test
Server has startup warnings:
Wed Mar 20 22:40:49.850 [initandlisten]
Wed Mar 20 22:40:49.850 [initandlisten] ** WARNING: Readahead for /data/db is set to 2048KB
Wed Mar 20 22:40:49.850 [initandlisten] ** We suggest setting it to 256KB (512 sectors) or less
Wed Mar 20 22:40:49.850 [initandlisten] ** http://dochub.mongodb.org/core/readahead
>
```
The fix for this turned out to be:
```
sudo blockdev --setra 256 /dev/md2
```
Where `/dev/md2` is the disk where the database files are stored.
