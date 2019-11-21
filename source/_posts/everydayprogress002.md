---
title: 每天进步一点点002-locust,pprof
author: ado
date: 2019-11-21 17:50:17
description: 每天进步一点点，locust的hello world
tags: [python, go, locust]
---

# Preface

看到有人在用locust，所以了解了一下，想起自己最近在做的一些东西，可以用来玩玩

# Content

### locust
```python
from locust import HttpLocust, TaskSet, between

def index(l):
    l.client.get("/")

def login(l):
    l.client.get("/login")#{"account":"testduke","password":"hehepassword"}

def register(l):
    l.client.get("/register")#{"account":"testduke","password":"hehepassword"}

class UserBehavior(TaskSet):
    tasks = {index:2,login:1,register:1}

class WebsiteUser(HttpLocust):
    task_set = UserBehavior
    wait_time = between(5.0,9.0)
```


### httpserver

```go
import (
    "net/http"
	_ "net/http/pprof"
)

func Tfunc TestNewHttpServer(t *testing.T) {
	server := NewHttpServer(":8888")
	server.AddHandler("/register",GET, handlers.Register)
	server.AddHandler("/login", GET,handlers.Login)
	server.AddHandler("/", GET,handlers.Index)
	go server.Start()
	//go func() {
		log.Println(http.ListenAndServe(":6060",nil))
	//}()
	sigs := make(chan os.Signal, 1)
	done := make(chan bool, 1)
	signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)
	go func() {
		sig := <-sigs
		log.Println()
		log.Println("Recieve Signal :", sig)
		done <- true
	}()
	<-done
	log.Println("Application terminated")
}
```
这个是golang写的测试代码，特别要注意的是
> 导入 pprof 包的时候会默认给httphandler注册pprof相关的函数，所以如果自己的业务http server先启动的话，会导致端口冲突，pprof相关接口用不了，所以要先启动pprof的http服务，再启动自己的业务http服务

## Result
最终效果图如下:  
![locust](./locust.png)  

# Reference
* [locust入门文档](https://docs.locust.io/en/stable/installation.html) : 包括了从入门到精通
* 