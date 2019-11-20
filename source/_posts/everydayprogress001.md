---
title: 每天进步一点点001-etcd
author: ado
date: 2019-11-12 10:34:01
description: 搭建etcd节点，使用golang连接到etcd，以etcd作为服务发现
tags: [go, etcd, docker]

---

# Preface

最近迷失了方向，所以每天强行学习一点点吧。

刚才总结了一下自己后续的规划，感觉以后可能是做点小生意，或者是写app的什么的，主要用golang，如果前端有可能的话，估计会用flutter。

今天就先从最近开始但是尚未结束的etcd开始吧，算是摸索和入门

# Content

## Docker安装单节点etcd

### Docker/etcd官方推荐运行方式

```yaml
docker run \
  -d \
  -p 2379:2379 \
  -p 2380:2380 \
  -p 4001:4001 \
  -p 7001:7001 \
  -v /data/backup/dir:/data \
  --name some-etcd \
  elcolio/etcd:latest \
  -name some-etcd \
  -discovery=https://discovery.etcd.io/blahblahblahblah \
  -advertise-client-urls http://192.168.1.99:4001 \
  -initial-advertise-peer-urls http://192.168.1.99:7001
```

需要注意的是，这里的discovery需要自己去调用官方提供的公共服务来获得自己特有的discovery，随便填会随便报错。当然，有条件的话你搭建自己的发现服务也可以：

```sh
curl https://discovery.etcd.io/new?size=3
```



### Code

最近是在看go语言高级编程看到这一节才想起之前搭建etcd集群一直都是失败的，测试代码如下：

```go
package advanced_go

import (
	"fmt"
	"github.com/zieckey/etcdsync"
)

func GoEtcd()  {
	m,err := etcdsync.New("/lock",10,[]string{"http://192.168.2.91:2379"})
	if m == nil || err != nil {
		fmt.Printf("etcdsync.New failed.\n %v \n",err)
		return
	}
	err = m.Lock()
	if err != nil {
		fmt.Printf("etcdsync.Lock failed.\v %v \n",err)
		return
	}
	fmt.Println("etcdsync.Lock OK")
	fmt.Println("Get the lock. Do something here.")

	err = m.Unlock()
	if err != nil {
		fmt.Printf("etcdsync.Unlock failed.\n %v \n",err)
	}else {
		fmt.Println("etcdsync.Unlock OK")
	}
}

```

测试结果如下：

> etcdsync.Lock failed. client: endpoint http://192.168.2.91:2379 exceeded header timeout 

使用 docker inspect看到的state部分的状况如下：

```json
"State": {
            "Status": "exited",
            "Running": false,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 0,
            "ExitCode": 1,
            "Error": "",
            "StartedAt": "2019-11-20T03:21:47.165684657Z",
            "FinishedAt": "2019-11-20T03:22:49.476739053Z"
        },

```

运行docker logs my-etcd看到如下如下结果：

> Using default CLIENT_URLS (http://0.0.0.0:4001,http://0.0.0.0:2379)
> Using default PEER_URLS (http://0.0.0.0:7001,http://0.0.0.0:2380)
> Running '/bin/etcd -data-dir=/data -listen-peer-urls=http://0.0.0.0:7001,http://0.0.0.0:2380 -listen-client-urls=http://0.0.0.0:4001,http://0.0.0.0:2379 -name my-etcd -discovery=https://discovery.etcd.io/0db6471c7333720cec321f6e85f3c1e8 -advertise-client-urls http://192.168.2.91:4001 -initial-advertise-peer-urls http://192.168.2.91:7100'
> BEGIN ETCD OUTPUT
> 2019/11/20 03:21:47 etcd: listening for peers on http://0.0.0.0:2380
> 2019/11/20 03:21:47 etcd: listening for peers on http://0.0.0.0:7001
> 2019/11/20 03:21:47 etcd: listening for client requests on http://0.0.0.0:2379
> 2019/11/20 03:21:47 etcd: listening for client requests on http://0.0.0.0:4001
> 2019/11/20 03:21:47 etcdserver: datadir is valid for the 2.0.1 format
> 2019/11/20 03:21:49 discovery: found self c482fe336fe84311 in the cluster
> 2019/11/20 03:21:49 discovery: found 1 peer(s), waiting for 2 more
> 2019/11/20 03:22:49 etcd: stopping listening for client requests on http://0.0.0.0:4001
> 2019/11/20 03:22:49 etcd: stopping listening for client requests on http://0.0.0.0:2379
> 2019/11/20 03:22:49 etcd: stopping listening for peers on http://0.0.0.0:7001
> 2019/11/20 03:22:49 etcd: stopping listening for peers on http://0.0.0.0:2380
> 2019/11/20 03:22:49 etcd: unrecognized HTTP status code 502

也就是启动就成功，我了个大槽，先放着。

# Reference

* [Docker hub/elcolio/etcd](https://hub.docker.com/r/elcolio/etcd)
* [cluter/public-etcd-discovery-service](https://github.com/etcd-io/etcd/blob/master/Documentation/v2/clustering.md#public-etcd-discovery-service)

# PS

docker image在docker hub都有详细说明，所以仔细阅读就可以了，不能心急