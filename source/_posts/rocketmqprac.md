---
title: Hello,RocketMQ
author: ado
date: 2019-06-08 17:01:00
description: 工作中需要用到RocketMQ,所以需要了解一下
tags: [rocketmq]
---

# Preface

[快速入门](<https://rocketmq.apache.org/docs/quick-start/>)

## 系统要求

* 64bit OS, 官方建议Linux/Unix/Mac
* JDK 1.8+
* Maven 3.2.x
* Git
* Broker服务需要 4g+ 的磁盘空间

## 下载与安装

[当前版本源码下载](<https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.4.0/rocketmq-all-4.4.0-source-release.zip>)

[发行版下载](<http://rocketmq.apache.org/release_notes/release-notes-4.4.0/>)

### 解压

```sh
 > unzip rocketmq-all-4.4.0-source-release.zip
 > cd rocketmq-all-4.4.0/
 > mvn -Prelease-all -DskipTests clean install -U
 > cd distribution/target/apache-rocketmq
```

### Maven的安装

从官方下载 [Maven的binary](<https://maven.apache.org/download.cgi>) ，直接解压。

将解压缩出来的 **/bin** 目录添加到 **~/.bash_profile** ，然后 *source ~/.bash_profile* 之后就可以使用 **mvn** 指令了。

## 启动Name Server

```sh
> nohup sh bin/mqnamesrv &
> tail -f ~/logs/rocketmqlogs/namesrv.log
  The Name Server boot success...
```

## 启动Broker

```sh
> nohup sh bin/mqbroker -n localhost:9876 &
> tail -f ~/logs/rocketmqlogs/broker.log 
  The broker[%s, 172.30.30.233:10911] boot success...
```

## 发送和接受消息

首先，得告诉终端 name server得地址。RocketMQ提供了很多方法来获取这个。最简单的是通过环境变量 `NAMESRV_ADDR` ：

```sh
 > export NAMESRV_ADDR=localhost:9876
 > sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
 SendResult [sendStatus=SEND_OK, msgId= ...

 > sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
 ConsumeMessageThread_%d Receive New Messages: [MessageExt...
```

## 关闭服务

```sh
> sh bin/mqshutdown broker
The mqbroker(36695) is running...
Send shutdown request to mqbroker(36695) OK

> sh bin/mqshutdown namesrv
The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK
```

## Golang的RocketMQ-client

[rocketmq-client-go](<https://github.com/apache/rocketmq-client-go>) 这个是目前github上最多star的项目 。

根据 [文档](<https://github.com/apache/rocketmq-client-go/blob/master/doc/Introduction.md>) 描述，他应该是需要依赖[rocketmq-client-cpp](https://github.com/apache/rocketmq-client-cpp) 的。