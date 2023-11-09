+++
title = "通过Rclone + 对象存储实现同步盘"
date = "2023-11-09T10:23:52+08:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["工具", "私有云"]
keywords = ["", ""]
description = "命令行比GUI方便多了"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

## 下载安装
https://rclone.org/downloads/
s
如果需要挂载为磁盘，windows下需要安装winfsp：https://winfsp.dev/rel/

## 配置cos：
rclone config

腾讯云：
https://rclone.org/s3/#tencent-cos

阿里云
https://rclone.org/s3/#alibaba-oss

## 使用（假设配置将存储环境命名为cos）：
列出桶：rclone lsd cos

本地同步COS：rclone sync local-folder cos:bucket-name/folder

COS同步本地：rclone sync cos:bucket-name/folder local-folder

同步前，建议使用 --dry-run 参数，确认哪些文件将会被删除和下载，避免不必要的资源浪费。

## 其他命令

```bash
rclone mkdir cos:rclone-test-1251668577，创建存储桶rclone-test-1251668577
rclone ls cos:rclone-test-1251668577，列出rclone-test-1251668577根目录下的文件
rclone copy local-folder/ cos:rclone-test-1251668577/，拷贝本地文件或目录到COS上，不会删除目的端的其他文件
rclone copy cos:rclone-test-1251668577 cos:rclone-test-backup-1251668577，同一个存储，在服务端使用copy操作拷贝文件
rclone sync local-folder/ cos:rclone-test-1251668577/ --backup-dir cos:rclone-test-backup-1251668577/20191011，将本地文件同步到cos，并备份被删除或修改的文件到备份存储桶中
rclone copy --max-age 24h --progress --no-traverse local-folder/ cos:rclone-test-1251668577/，--max-age 24h过滤出来最近24小时变更过的文件，--progress显示拷贝进度，--no-traverse在从源拷贝少量文件到目的中大量目的文件时，速度会更快
rclone check local-folder/ cos:rclone-test-1251668577/ --one-way，查看本地文件是否都同步到了目的端，默认校验修改时间和大小
rclone --min-size 500B lsl  cos:rclone-test-1251668577/，查看存储桶中500B以上的文件列表
rclone --dry-run --min-size 300B delete cos:rclone-test-1251668577/，查看存储桶中500B以上的待删除文件列表
rclone delete oss:oss-test-bucket-1215715707/ --include=/stl-views.gdb，删除根目录下的stl-views.gdb文件，如果不带/前缀，则会删除所有stl-views.gdb文件
rclone size cos:rclone-test-1251668577/，查看存储桶中对象数目和占用的空间大小
rclone mount cos:rclone-test-1251668577/ rclone-mnt/，将cos挂载成一个本地文件系统
rclone ncdu cos:rclone-test-1251668577/，一个简易文本形式的文件浏览器，用于存储桶中的文件浏览、文件和文件夹删除等操作
rclone cat cos:rclone-test-1251668577/test.cpp --head 10，输出test.cpp的前10个字节
echo "hello world" |rclone rcat cos:rclone-test-1251668577/rcat.txt将标准输出复制到存储桶的rcat.txt文件中，会覆盖目标文件
rclone sync oss:oss-test-bucket-1215715707/  cos:rclone-test-1251668577/ -P，同步oss存储桶中的数据到cos存储桶中，-P选项显示进度
rclone check oss:oss-test-bucket-1215715707/  cos:rclone-test-1251668577/ -P，进行数据对比校验
rclone md5sum cos:rclone-test-1251668577/，为所有文件生成MD5值
rclone tree cos:rclone-test-1251668577/ -C -D，显示文本格式的目录树结构，-C选项带颜色显示，-D显示上次修改时间
```

## 参考资料
https://cloud.tencent.com/developer/article/1520867

https://rclone.org/