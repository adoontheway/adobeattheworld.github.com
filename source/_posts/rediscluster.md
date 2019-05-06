---
title: Redis 连接问题
author: ado
date: 2019-05-06 13:49:06
description: 连接Redis Cluster遇到问题汇总
tags: [docker,redis,go]
---



## Preface

最近在搭游戏框架，需要用 **redis** 来存一些缓存数据。

于是就打算利用之前搭建好的 **redis docker**容器。

[Docker搭建mysql和Redis集群](/dockerrediscluster.md)

进展不是很顺利，所以把过程记录下来。

## Code

代码方面使用的是 **golang**。

连接的三方库是 [gomodule/redigo](https://github.com/gomodule/redigo/)

测试代码如下：

```go
package test

import (
	"fmt"
	"github.com/gomodule/redigo/redis"
	"testing"
	"time"
)

func TestRedis(t *testing.T){
	readTimeOut := redis.DialReadTimeout(time.Second*3)
	connectTimeOut := redis.DialConnectTimeout(time.Second*10)
	writeTimeOut := redis.DialWriteTimeout(time.Second*3)
	conn, err := redis.Dial("tcp","192.168.2.127:7001",readTimeOut,connectTimeOut,writeTimeOut)
	if err != nil {
		 fmt.Println("Dial Failed:",err)
		 t.FailNow()
	}
	err = conn.Send("SET","foo","bar")
	if err != nil {
		fmt.Println("Set Failed:",err)
		t.FailNow()
	}
}

```



## Steps



首先遇到的是这个问题：

*connectex: No connection could be made because the target machine actively refused it.*

然后去**redis**里面查看日志。

```sh
sudo docker exec -it redis-master bash
```

发现**docker**里面没有**vi**之类的文本编辑器查看日志，通过以下命令安装**vim**。

```sh
apt-get update
apt-get install vim
```

遇到了：*Temporary failure resolving 'deb.debian.org'*

```shell
chmod o+r /etc/resolv.conf
```

然后重新运行安装**vim**的命令就好了。

*/usr/local/log/redis.log* 里面其实是没有连接信息的。

在 **redis-master** 容器中运行:

```sh
redis-cli
```

发现居然连不上了，才发现是没有启动 **redis-server** ，乌龙一个。

**docker**的**redis**容器中找不到ps命令，哎。

```sh
apt-get install procps
```

然后先后通过修改代码中的ip地址为master的地址

```sh
redis.Dial("tcp","172.17.0.2:6379",readTimeOut,connectTimeOut,writeTimeOut)
```

报了 <u>i/o timeout</u> 。

仔细看了一下启动命令，发现是没有做主机到容器之间的端口映射。

于是

```sh
docker commit redis-master redis-master
```

提交备份master

```sh
docker run -p 7001:6379 -td redis-master
```

之后就可以使用主机地址连接上了。

大乌龙。。。。