---
title: Redis Dockerfile
author: ado
date: 2019-05-08 16:52:25
description: Redis还是有些问题
tags: [redis, docker]
---



# Preface

今天使用密码连接**reids**的时候发现报错，注释掉 **AUTH** 相关代码后居然可以连接;

在  **redis-cli** 中发现 **config** 为空;

使用 *ps -ef | grep redis* 中发现 **redis-server** 的启动参数居然是默认的，而不是我们指定的 */usr/local/etc/redis/redis.conf* 文件;

加上之前主从结构并没有生效，所以现在必须要重新整理一遍了。

 # Step

## 重启使用 redis.conf

在进程里面看到的 **redis-server** 的进程 **id** 是 <u>1</u>，在 **kill** 或者在 **redis-cli** 中使用 **shutdown** 会导致直接退出此容器。

在 **redis-cli** 中输入 *help @server* 的时候，发现其中有个命令叫做 *CONFIG SET* ，其实也就是更新 **redis.conf** 里的那些设置。

例如 *CONFIG SET requirepass 12345*这样的。

结果：失败

## 重新做一个镜像吧

之前那样傻呼呼的运行命令行，不是很方便，对于后面运维的啥的也不是很好。

所以就使用 Dockerfile 吧。

```dockerfile

FROM redis:lastest

# MAINTAINER

RUN echo > Starting update and install dev dependencies.

RUN apt-get update && apt-get install -y vim procps

RUN echo > Update and install finished.

EXPOSE 6379

ADD ./data/redis.conf /usr/local/etc/redis/redis.conf
ADD ./data/master.log /usr/local/log/redis.log

# CMD 

RUN echo > Starting redis-server.

ENTRYPOINT ["redis-server","/usr/local/etc/redis/redis.conf"]

RUN echo > Redis-server start completed.

```

由于公司流量宝贵，所以先备份到这里，这个镜像有点问题，起不来，后续更新。

# Reference

[dockerfile介绍](http://www.dockerinfo.net/dockerfile%E4%BB%8B%E7%BB%8D)