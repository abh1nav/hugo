+++
title = "Homebrew and Python-MySQL"
date = "2013-08-13T17:05:12-04:00"
tags = ["devops", "python", "mysql"]
type = "post"
aliases = [
    "/blog/2013/08/13/homebrew-and-python-mysql/"
]
+++

To be able to pip install the python mysql library on OS X, you need mysql client installed locally. If you have no need for the full mysql package, here’s how to get it working:<!--more-->
```bash
brew install mysql --client-only --universal
pip install MySQL-python
```
