---
title: 每天进步一点点006-etcd windows集群环境
author: ado
date: 2019-12-14 20:58:53
description: 今天尝试在windows上搭建etcd集群环境
tags: [etcd]
---
# Preface
之前的环境比较复杂：我主机是windows，centos是vmware里面的虚拟机，docker又是centos里面的。  
之前是打算在基于centos的docker搭建一个etcd本地集群来开发用，用作测试分布式服务发现，当然也可以用作分布式数据存储。  
分布式数据存储也可以使用redis，在分布式数据存储这一点上etcd和redis各自的优缺点是啥，需要后面调查和验证一下。  
在此略过。

# Contents
分布式服务最少都是3个起步，都是奇数个，以匹配raft选举策略，所以需要配置3个服务。
## 配置文件
以下分别是1，2，3号etcd服务的配置文件：
```yml
name: etcd01
data_dir: .\data\etcd01 
advertise-client-urls: http://127.0.0.1:2379 
listen-client-urls: http://127.0.0.01:2379 
listen-peer-urls: http://127.0.0.1:2380 
initial-advertise-peer-urls: http://127.0.0.1:2380 
initial-cluster-token: etcd-cluster-1 
initial-cluster: etcd01=http://127.0.0.1:2380,etcd02=http://127.0.0.1:2381,etcd03=http://127.0.0.1:2382 
initial-cluster-state: new 
```

```yml
name: etcd02
data_dir: .\data\etcd02 
advertise-client-urls: http://127.0.0.1:3379 
listen-client-urls: http://127.0.0.01:3379 
listen-peer-urls: http://127.0.0.1:2381 
initial-advertise-peer-urls: http://127.0.0.1:2381 
initial-cluster-token: etcd-cluster-1 
initial-cluster: etcd01=http://127.0.0.1:2380,etcd02=http://127.0.0.1:2381,etcd03=http://127.0.0.1:2382 
initial-cluster-state: new 
```

```yml
name: etcd03
data_dir: .\data\etcd03 
advertise-client-urls: http://127.0.0.1:4379 
listen-client-urls: http://127.0.0.01:4379 
listen-peer-urls: http://127.0.0.1:2382 
initial-advertise-peer-urls: http://127.0.0.1:2382 
initial-cluster-token: etcd-cluster-1 
initial-cluster: etcd01=http://127.0.0.1:2380,etcd02=http://127.0.0.1:2381,etcd03=http://127.0.0.1:2382 
initial-cluster-state: new 
```

## 启动脚本

```bat
etcd --config-file=F:/software/etcd-v3.4.0-windows-amd64/etcd01.yml
pause
```

## 启动结果
![启动结果](./etcd1.png)

# Conclusion
整个过程不是很顺利，首先是按照官方的文档进行操作，windows会有奇怪的问题，于是改成了配置文件操作；  
接着是windows的权限问题，整得很头疼；  
现在是3个etcd服务器都跑起来了，但是etcdctl又用不了；  
所以先就此备份。