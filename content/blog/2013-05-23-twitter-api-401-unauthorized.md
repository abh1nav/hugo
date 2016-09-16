+++
title = "Twitter API: 401 Unauthorized"
date = "2013-05-23T19:30:00-04:00"
tags = ["programming", "twitter"]
type = "post"
aliases = [
    "/blog/2013/05/23/twitter-api-401-unauthorized/"
]
+++

**Symptom**: Twitter API returns a 401 Unauthorized when you start the OAuth process by obtaining a bearer token. This can happen all of a sudden possibly breaking existing processes that were working.  
<br>
{{% screenshot "/images/2013-05-23-1.jpg" %}}
<br><br>
**Problem**: The most likely culprit is the clock on your server. If this gets out of sync (even by as little as 20s), Twitter’s amazing API will barf and return a helpful _“401 Unauthorized”_
  
  
**Solution**:
```
sudo ntpdate ntp.ubuntu.com
```
