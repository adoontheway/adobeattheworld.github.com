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

[Docker搭建mysql和Redis集群](https://adobeattheworld.github.io/2019/05/04/dockerrediscluster/)

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

然后先后通过修改代码中的**ip**地址为**master**的地址

```sh
redis.Dial("tcp","172.17.0.2:6379",readTimeOut,connectTimeOut,writeTimeOut)
```

报了 <u>i/o timeout</u> 。

仔细看了一下启动命令，发现是没有做主机到容器之间的端口映射。

于是

```sh
docker commit redis-master redis-master
```

提交备份**master**

```sh
docker run -p 7001:6379 -td redis-master
```

之后就可以使用主机地址连接上了。

大乌龙。。。。

### 设置密码

给 **master** 设置密码，修改 **conf** 中的 **requirepass** 。

然后将 **master** 的配置和 log 文件都放到新建的文件夹 **masterdata** 中，

然后运行 修改masterdata的权限，不然后面会报访问拒绝之类的:

```sh
chmod -R 777 masterdata
```

然后用以下方式启动:

```sh
sudo docker run --name redis-master --privileged=true -p 7001:6379 -v $PWD/masterdata:/data -v $PWD/masterdata/master.log:/var/log/redis/redis.log -d redis redis-server /data/redis-master.conf

```

有报错，使用 *sudo docker logs redis-master* 查看容器日志，发现：

```sh
*** FATAL CONFIG FILE ERROR ***
Reading the configuration file, at line 174
>>> 'logfile "/usr/local/log/redis.log"'
Can't open the log file: No such file or directory
```

这是读取 **redis-master.conf** 第174日志文件配置出的错，也就是上面命令行的日志文件映射出错了。

最终的运行命令如下：

```sh
sudo docker run -d -v $PWD/masterdata/redis-master.conf:/usr/local/etc/redis/redis.conf -v $PWD/masterdata/master.log:/usr/local/log/redis.log -p 7001:6379 --name redis-master redis
```

最后跑一遍:

```sh
docker update --restart=always redis-master
```

重启，启动成功之后执行：

```sh
sudo docker ps
```

**redis-master**赫然在列。

**redis-slave**同理。

