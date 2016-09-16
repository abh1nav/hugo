+++
title = "pip install scipy on Ubuntu"
date = "2013-09-19T01:17:29-04:00"
tags = ["programming", "python", "scipy"]
type = "post"
aliases = [
    "/blog/2013/09/19/pip-install-scipy-on-ubuntu/"
]
+++

On Ubuntu 12.04 Server, a pip install scipy barfs with
<pre>
library dfftpack has Fortran sources but no Fortran compiler found
</pre>
because it wants you to
```bash
sudo apt-get install libamd2.2.0 libblas3gf libc6 libgcc1 \
libgfortran3 liblapack3gf libumfpack5.4.0 libstdc++6 \
build-essential gfortran python-all-dev \
libatlas-base-dev
```
and,
```bash
pip install numpy
```
before you try to
```bash
pip install scipy
```
