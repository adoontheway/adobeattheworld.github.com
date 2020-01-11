---
title: 每天进步一点点013 - Colly
author: ado
date: 2020-01-11 20:33:51
description: 利用colly爬取网站
tags: [go, colly]
---

# Preface
嗯，最近在看一些小说，然后想到之前看过 **go** 有 **colly** 的爬虫框架存在，于是就来试试。  
反正自己一闲下来就发慌。  

# Contents
目前只是爬到小说列表然后存放到本地 **mongo** 中，然后通过页面的下一页按钮来翻页。  
当前只爬了 *120秒* 的数据，也爬了几百条。  
## Code
```go
package main

import (
	"context"
	"fmt"
	"github.com/gocolly/colly"
	"github.com/gocolly/colly/debug"
	"github.com/zolamk/colly-mongo-storage/colly/mongo"
	"go.mongodb.org/mongo-driver/bson"
	mongo2 "go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
	"time"
)

type Book struct {
	Title string `json:"title"`
	Author string 	`json:"author"`
	Tag string	`json:"tag"`
	Hot string	`json:"hot"`
	Chars string `json:"chars"`
} 

const (
	ROOT_URL  = "http://xsw.xyz/shuku/"
	MONGO_URI = "mongodb://localhost:27017"
)
var coll *mongo2.Collection
func init()  {
	client, err := mongo2.NewClient(options.Client().ApplyURI(MONGO_URI))
	if err != nil {
		panic(err)
	}
	ctx,_ := context.WithTimeout(context.Background(), 10*time.Second)
	err = client.Connect(ctx)
	if err != nil {
		panic(err)
	}
	coll = client.Database("xsw").Collection("book_list")
}

func main()  {
	c := colly.NewCollector(
		colly.Debugger(&debug.LogDebugger{}),
		colly.AllowedDomains("xsw.xyz"),
		colly.CacheDir("./xsw"),
		colly.UserAgent("xsw"),
		colly.Async(true),
		)
	storage := &mongo.Storage{
		Database:"books_xsw",
		URI:MONGO_URI,
	}
	if err := c.SetStorage(storage); err != nil {
		panic(err)
	}
	c.Limit(&colly.LimitRule{
		DomainGlob:"xsw.*,",
		Parallelism:2,
		RandomDelay:3*time.Second,
	})
	detailCollector := c.Clone()
	c.OnHTML(".container .mod .bd", func(e *colly.HTMLElement) {
		fmt.Println("found list")
		e.ForEach("ul .column-2", func(_ int, el *colly.HTMLElement) {
			title := el.ChildText(".right .name")
			author := el.ChildText(".info .author")
			words := el.ChildText(".info .words")
			update := el.ChildText(".info font")
			ctx,_ := context.WithTimeout(context.Background(), 3*time.Second)
			_,err := coll.InsertOne(ctx,bson.M{"title":title,"author":author,"words":words,"update":update})
			if err != nil {
				fmt.Println(err)
			}
		})
	})
	c.OnHTML(".nextPage", func(e *colly.HTMLElement) {
		e.Request.Visit(e.Attr("href"))
	})
	c.OnError(func(response *colly.Response, e error) {
		fmt.Println("Request URL",response.Request.URL,", failed with response:",response,"\nError:",e)
	})
	detailCollector.OnHTML("div[mod block book-all-list]", func(e *colly.HTMLElement) {
		
	})
	c.Visit(ROOT_URL)
	time.Sleep(120*time.Second)
}
```

## Description
主要是通过读取小说的书库的第一页，解析单页的小说列表中的内容，存入 **mongo** ，然后访问下一页。  
需要注意的是页面元素的筛选需要去复习一遍 **jquery** 的 **selector** 。   

# Reference
* [colly](https://github.com/gocolly/colly/blob/master/cmd/colly/colly.go)
* [jQuery操作Select](https://www.cnblogs.com/shanyou/archive/2011/07/11/2103422.html)
* [mongo-go-driver](https://github.com/mongodb/mongo-go-driver)
* [go colly examples](http://go-colly.org/docs/examples)