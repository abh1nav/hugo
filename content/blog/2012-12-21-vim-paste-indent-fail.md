+++
title = "Vim won't let you paste properly?"
date = "2012-12-21T00:45:03-05:00"
tags = ["vim"]
type = "post"
aliases = [
    "/blog/2012/12/21/vim-paste-indent-fail/"
]
+++

When you paste in vim, if the indents look funny, then before you paste, run:<!--more-->
```vim
:set paste
```
Before pasting, ensure vim is in insert mode. Once pasting is done, revert back to normal mode with
```vim
:set nopaste
```
and continue editing.