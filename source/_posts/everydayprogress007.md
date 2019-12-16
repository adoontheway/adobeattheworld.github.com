---
title:  每天进步一点点007-golang中使用etcd
author: ado
date: 2019-12-16 11:01:10
description: 在golang中使用etcd做服务发现
tags: [etcd, go]
---

# Preface
当前在技术栈中使用zookeeper作服务发现，zookeeper是java的，对java比较友好，如果后续以golang作为自己的职业方向的话，可以选择etcd。  
上一节已经在windows上搭建好了一个比较简单的etcd集群，这里就在golang中用他。

#  Content

## Codes
```go
// proxyserver.go
import (
	"bytes"
	"context"
	"crypto/rand"
	"encoding/binary"
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"log"
	"net"
	"os"
	"time"
)
var etcdClient *clientv3.Client

func StartProxy(port int32) {
	log.SetFlags(log.Lshortfile | log.Ldate | log.Ltime)
	f,err := os.Create(fmt.Sprintf("./logs/proxy-%d.log",port))
	if err != nil {
		fmt.Println("Error when creating log file:",err)
		os.Exit(1)
	}

	log.SetOutput(f)
	l, err := net.Listen("tcp", fmt.Sprintf("0.0.0.0:%d",port))
	if err != nil {
		log.Println("Error Listening:", err)
		os.Exit(1)
	}
    resolveLocalIp()
	startEtcd()
	getServers()
	watchServers(l.Addr().String())

	defer func() {
		l.Close()
		etcdClient.Close()
	}()

	for {
		conn, err := l.Accept()
		if err != nil {
			log.Println("Error accepting:", err)
			os.Exit(1)
		}
		log.Printf("Receivied message %s -> %s \n", conn.RemoteAddr(), conn.LocalAddr())
		go handleRequest(conn)
	}
}


func startEtcd()  {
	var err error
	etcdClient, err = clientv3.New(clientv3.Config{
		Endpoints:[]string{"localhost:2379","localhost:3379","localhost:4379"},
		DialTimeout:5*time.Second,
	})
	if err != nil {
		log.Fatalln("connect failed, err:",err)
		return
	}
	log.Println("connect successed")
}

func getServers()  {
	ctx, cancel := context.WithTimeout(context.Background(),  time.Second*3)
	resp, err := etcdClient.Get(ctx, "servers/proxyservers/")
	cancel()
	if err != nil {
		log.Println("get failed, err:",err)
		return
	}
	for _,ev := range resp.Kvs {
		log.Printf("%s : %s \n", ev.Key, ev.Value)
	}
}

func watchServers(addr string)  {
	etcdClient.Put(context.Background(),"servers/proxyservers/", addr)
	for {
		rch := etcdClient.Watch(context.Background(), "servers/proxyservers/")
		for wresp := range  rch {
			for _, ev := range wresp.Events {
				log.Printf("Event[type:%s key:%q value:%q] \n", ev.Type, ev.Kv.Key, ev.Kv.Value)
			}
		}
	}
}

func resolveLocalIp()  {
	ifaces, err := net.Interfaces()
	if err != nil {
		log.Fatalln("Get net interface error:",err)
		return
	}
	for _,i := range ifaces {
		addrs, err := i.Addrs()
		if err != nil {
			log.Println("Loop net interface error:",err)
			continue
		}
		for _, addr := range addrs {
			var ip net.IP
			switch v := addr.(type) {
			case *net.IPNet:
				ip = v.IP
			case *net.IPAddr:
				ip = v.IP

			}
			log.Println("Get local ip address:", ip.String())
		}
	}
}
```

```go
// proxyserver_test.go
func TestStartProxy(t *testing.T) {
	StartProxy(8889)
}
func TestStartProxy2(t *testing.T) {
	StartProxy(8890)
}
```

## Result
以下是8889的日志：
> 2019/12/16 11:16:34 proxyserver.go:244: Get local ip address: fe80::e932:a5a7:a3ef:45f6
2019/12/16 11:16:34 proxyserver.go:244: Get local ip address: 192.168.1.12
...
2019/12/16 11:16:34 proxyserver.go:244: Get local ip address: ::1
2019/12/16 11:16:34 proxyserver.go:244: Get local ip address: 127.0.0.1
2019/12/16 11:16:34 proxyserver.go:195: connect successed
2019/12/16 11:16:34 proxyserver.go:207: servers/proxyservers/ : [::]:8890 
2019/12/16 11:18:56 proxyserver.go:217: Event[type:PUT key:"servers/proxyservers/" value:"[::]:8890"] 
