+++
title = "My Docker Cheatsheet"
date = "2023-11-09T17:19:34+08:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["", ""]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

mysql
https://zhuanlan.zhihu.com/p/158063202
https://hub.docker.com/_/mysql/

redis

## Docker性能损失

[V2EX讨论](https://www.v2ex.com/t/394313)

> CPU 基本不会有损失。
> 
> 损失只出现在 I/O 上，包括网络 I/O 和文件系统 I/O。这两者就要看不同的实现了。
> 
> ......但是对于 99% 的应用程序来说，根本不需要考虑这个损失……