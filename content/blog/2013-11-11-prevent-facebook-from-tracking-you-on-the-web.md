+++
title = "Prevent Facebook from tracking you on the Web"
date = "2013-11-11T12:51:00-05:00"
tags = ["linux", "tinfoil hat"]
type = "post"
aliases = [
    "/blog/2013/11/11/prevent-facebook-from-tracking-you-on-the-web/"
]
+++

A simple search for “facebook like button tracking you” turns up a bunch of articles and blog posts about how Facebook is creeping on you wherever you go; even if you’re not signed in or actively clicking their ubiquitous _like_ button ([example](http://bit.ly/1fwCi5i)). I’ve come up with a simple solution to the problem.  <!--more-->

*Caveat*: This will prevent you from being able to use Facebook.  

**Step 1**: Add the following entries to your hosts file

```bash
127.0.0.1 facebook.com
127.0.0.1 www.facebook.com
127.0.0.1 connect.facebook.net
```

If you’re not familiar with editing your hosts file, [here’s a tutorial](http://bit.ly/17jkvci).  

**Step 2**: Reboot  

And done!