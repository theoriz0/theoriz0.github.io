+++
title = "Grep 命令检索代码"
date = "2023-11-15T11:59:46+08:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["Shell", ""]
keywords = ["", ""]
description = "快速找到其他项目中的代码参考范例。"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

## Grep 常用选项介绍

-  `-r`，递归查找
    
-  `-n`，搜索结果显示行号
    
-  `-i`，忽略大小写
    
-  `-v`，反向匹配
    
-  `-w`，匹配整个单词
    
-  `-E`，匹配扩展的正则表达式

- `-l`，只打印匹配的文件名。

## 递归查找并显示行号

```bash
grep -rn memcpy
grep -rni memcpy #忽略大小写
```

## 限制文件类型或其它条件

```bash
grep -rn memcpy | grep -v .o.cmd #排除.txt文件
grep -rn memcpy | grep -Ev '\.txt\.md|\.log|\.yaml' #使用正则表达式排除多种类型
grep -rn --exclude=*.txt memcpy #不在.txt中搜索
grep -rn --include=*.go memcpy #只在.go中搜索
```

## 限制目录

```bash
grep -rn --exclude-dir=out memcpy
grep -rn --exclude-dir=out --exclude-dir=doc memcpy
```

## 匹配整个单词

```bash
grep -rnw memcpy
```

如果想过滤`zmemcpy`或者`memcpyl`，但是想保留`MCD_memcpy`, `memcpy_16`，需要写正则表达式。

## 查看查找结果的上下文

```bash
grep -rn -B 3 -A 2 MEMCPY #匹配的前三行+后两行
```

## grep 配合 find

```bash
find . -iname Makefile -o -iname *.inc -o -iname *.mk | xargs grep -rn CFLAGS
```

管道实现的是将前面的输出stdout作为后面的输入stdin，但是有些命令不接受管道的传递方式。例如：ls，这是为什么呢？因为有些命令希望管道传递过来的是参数，但是直接使用管道有时无法传递到命令的参数位。这时候就需要xargs，xargs实现的是将管道传递过来的stdin进行处理然后传递到命令的参数位置上。

## 参考资料

[使用grep搜索代码的几个示例](https://blog.csdn.net/guyongqiangx/article/details/70161189)

[Linux那点事-xargs命令详解](https://www.jianshu.com/p/676353506f0b)