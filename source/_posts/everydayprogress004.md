---
title: 每天进步一点点004-httprouter middleware
author: ado
date: 2019-11-29 11:17:44
description: 今天其实是要接着整nginx的https和wss的，昨天的有两个新的知识点需要在这里备份一下
tags: [go, httprouter, cors, middleware]
---

# Preface
昨天整 **nginx** 代理 **wss** 的事情。  
没有前端，自己把前端的发布版本搞来部署到 **nginx** 上；  
没有api服务，所以自己用 **golang** 写了个代理服务器，将前端的 **api** hack到我的代理服务器来，
然后这里可以灵活的代理某个服务器，顺便用 **nginx** 代理一下此端口，实现 *https* 支持。  
话说 **js** 真心脆弱.....  

# Content

```go
package apiserver

import (
	"fmt"
	"github.com/julienschmidt/httprouter"
	"github.com/rs/cors"
	"io"
	"io/ioutil"
	"net/http"
)

const target_api string = "http://192.168.1.91:8088"

func Start(port int32)  {
	router := httprouter.New()
	router.GET("/GameHandle",apiHandler)
	router.GET("/TokenHandle",apiHandler)
	handler := cors.Default().Handler(router)
	http.ListenAndServe(fmt.Sprintf(":%d",port),handler)
}

func apiHandler(w http.ResponseWriter,r *http.Request, ps httprouter.Params)  {
    realUrl := target_api+r.RequestURI
    fmt.Println(realUrl)
    resp,err := http.Get(realUrl)
    if err != nil {
        fmt.Fprint(w,err)
    }
    defer resp.Body.Close()
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Fprint(w,err)
    }
    w.Header().Set("Content-Type","text/plain; charset=utf-8")
    w.Header().Set("X-Powered-By","Golang/httprouter")
    w.Header().Set("Content-Length",fmt.Sprintf("%d",len(body)))
    //w.WriteHeader(http.StatusOK)
    io.WriteString(w,string(body))
    fmt.Print(string(body))
}

```
## cors问题
以上使用了 **rs/cors** 解决了跨域访问的问题，实际上你可以可以通过 *ResponseWriter().Header()* 去加入支持跨域访问的几个头，例如 **Access-Alow-Domain** 等等。  

# Reference
* [cors middleware](https://github.com/rs/cors)
* [net/http.ResponseWriter](https://golang.org/pkg/net/http/#ResponseWriter)