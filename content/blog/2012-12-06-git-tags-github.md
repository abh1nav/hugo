+++
title = "Git, Tags and GitHub"
date = "2012-12-06T05:06:47-05:00"
tags = ["git", "github"]
type = "post"
aliases = [
    "/blog/2012/12/06/git-tags-github/"
]
+++
  
Add a tag:  <!--more--> 
  
<pre>
git tag -a v0.1 -m "Version 0.1 Stable"
</pre>
  
Push tags to GitHub: 
<pre>
git push origin --tags
</pre>
  
Delete tag locally:  
  
<pre>
git tag -d v0.1
</pre>
  
Push to GitHub:  

<pre>
git push origin --tags
</pre>
    
    
<br>
Rinse.  
  
Repeat.  
