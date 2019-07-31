---
title: Redis操作原子性测试
author: ado
date: 2019-07-31 09:47:18
description: 最近需要在分布式系统中用到Redis存储和操作一个公有的值
tags: [go,redis]
---

# Preface

最近在一个分布式系统中多个服务器需要同步操作和使用一个公有值，思前想后，还是通过Redis来存储和操作这个值最方便了。于是决定对Redis的数据操作做一下调查。

# Content

## 前提

[is-redis-hincrby-atomic](<https://stackoverflow.com/questions/38954590/is-redis-hincrby-atomic>)

Stackoverflow大神说的：

> Since Redis is single-threaded, everything is atomic.

由于Redis是单线程的，所以他的所有操作都是原子操作，所以不存在并发读/写的问题。

且系统的redis不是集群的，只是普通的主从而已，所以不必担心落地的数据和redis中的数据有误差。

## 测试

放心起见，还是跑下测试代码心里有个底。

测试使用的redis库： [gomodule/redigo](<https://github.com/gomodule/redigo>)

参考：[examples-redigo](<https://github.com/pete911/examples-redigo>)

测试方法：

* 建立一个redis pool
* 利用pool去向hashset的一个字段进行1000次加1操作和500次减操作
* 结果如果等于500那就是正确的

```go
package test

import (
	"fmt"
	"github.com/gomodule/redigo/redis"
	"sync"
	"testing"
	"time"
)

var pool *redis.Pool

func init()  {
	pool = &redis.Pool{
		Dial: func() (conn redis.Conn, e error) {
			c,err := redis.Dial("tcp","192.168.1.20:7000")
			if err != nil {
				return nil, err
			}
			if _,err = c.Do("AUTH","11223344");err != nil {
				c.Close()
				return nil, err
			}
			if _,err := c.Do("SELECT",0); err != nil {
				c.Close()
				return nil,err
			}
			return c, nil
		},
		TestOnBorrow: func(c redis.Conn, t time.Time) error {
			if time.Since(t) < time.Minute {
				return nil
			}
			_,err := c.Do("PING")
			return err
		},
		MaxIdle:3,
		IdleTimeout:time.Minute*10,
	}
}

func TestHincryby(t *testing.T)  {
	conn := pool.Get()
	conn.Do("DEL","a","b")
	defer conn.Close()
	//测试用
	//arr,err := redis.Values(conn.Do("KEYS","*"))
	//leng := len(arr)
	//for i := 0; i < leng; i++{
	//	key,_ := redis.String(arr[i],nil)
	//	fmt.Println(key)
	//}
	wg := sync.WaitGroup{}
	wg.Add(1500)
	for i := 0; i < 1000; i++ {
		go incr(&wg)
		if i < 500 {
			go decr(&wg)
		}
	}
	wg.Wait()
	fmt.Println("Finished")
	reply,err := redis.String(conn.Do("HGET","a","b"))
	if err == nil {
		fmt.Println("Result:",reply)
	}else {
		fmt.Println(err)
	}
	pool.Close()
}

func incr(wg *sync.WaitGroup)  {
	conn := pool.Get()
	defer conn.Close()
	conn.Do("HINCRBY","a","b",1)
	wg.Done()
}
func decr(wg *sync.WaitGroup)  {
	conn := pool.Get()
	defer conn.Close()
	conn.Send("HINCRBY","a","b",-1)
	wg.Done()
}
```

redis是之前在centos的docker上安装的，映射的主机端口就是7000，而不是6379；

### 遇到的问题

* 第一个是自己犯傻，脑抽在Pool的dial里面SELECT DB是2，然后自己在docker里面默认使用的是0，所以查不到数据
* 一开始在进行测试前用代码去设置默认值： `conn.Do("HSET","a","b")` 后面的 HINCRBY 是死活操作不成功也不报错，最终结果一直是0，然后更换思路，直接在进行测试前删除这个字段就好了，估计是因为REDIS里面存的是string，而不是number所致，暂时没有时间深究。

## 结果

> === RUN   TestHincryby
> Finished
> Result: 500
> --- PASS: TestHincryby (3.05s)
> PASS
>
> Process finished with exit code 0

跑了几遍，测试通过。

# 备注

另外打算利用 [ltrim](<https://redis.io/commands/ltrim>) 与 [lpush](<https://redis.io/commands/lpush>) 实现一个定常队列。