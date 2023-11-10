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

## mysql

安装

```bash
docker run --name some-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:latest #tag 见docker hub
docker exec -ti some-mysql /bin/bash ## 进入docker镜像
mysql -u root -p
```

允许任何客户端连接

```sql
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
```

参考：

[](https://zhuanlan.zhihu.com/p/158063202)

[](https://hub.docker.com/_/mysql/)

redis

```bash
docker run -p 6379:6379 --name myredis -d redis:7.0.12  --requirepass my-secret-pw
```

wsl下，会遇到提示port already in use，而并不能查到端口被占用，可以将外部端口改为16379。

查看windows端口占用范围 [来源](https://cloud.tencent.com/developer/article/2006079)：
```bash
netsh interface ipv4 show excludedportrange protocol=tcp
```

进入redis

```bash
docker exec -it myredis bash 
redis-cli
auth my-secret-pw
```

## 自动重启
`docker run --restart-always` 

## 查看日志
```bash
docker logs myredis # 后面跟容器名 or 容器ID 都可以
docker logs --since 30m <容器名> # --since 30m 是查看此容器30分钟之内的日志情况。
```

## Docker性能损失

[V2EX讨论](https://www.v2ex.com/t/394313)

> CPU 基本不会有损失。
> 
> 损失只出现在 I/O 上，包括网络 I/O 和文件系统 I/O。这两者就要看不同的实现了。
> 
> ......但是对于 99% 的应用程序来说，根本不需要考虑这个损失……