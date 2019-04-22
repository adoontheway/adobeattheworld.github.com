## golang高级编程技巧

https://github.com/chai2010/advanced-go-programming-book/blob/master/ch5-web/ch5-03-middleware.md
## 其他
* 本地包无法导入，需要将本地的路径添加为gopath，这个路径必须是标准go环境的路径，导入方式按照路径导入即可: _import "gogogo/dome/lib"_
## 名词
* QPS : Query Per Second 每秒查询量，这个针对的是服务器上数据的读取操作
* TPS : Transaction Per Second 每秒事务处理量 ，这个针对的是服务器上数据的写入和修改操作
* 这两个指标只是参考，实际还有其他参考指标，平均无故障时间，负载持续时间，以及软件的系统资源使用情况
* 载荷处理超时时间 : 评判软件正确性的重要标准
* 如果某个api持续承受高负载时间过长，就需要考虑api设计是否合理，软件系统是否需要拆分
这一部分关于软件的评判标准来自于本书地200页。


## time
```go
//计时器，timer.c:chan<-是一个(chan int ,1）reset并不会清理这里的缓存，所以一定要确保接收
//如果没有即使清理的话会导致后续要写入的被丢弃
time.NewTimer(timeout)
//阻塞当前线程直到当前定时器结束
time.Sleep(time.Millisecond)
//返回timer对象，timer对象主要是使用timer.C通道来通知时间到了的
time.After(timer.Second)
```
```go
//方法和Timer差不多,Ticker.C会在计时器到点的时候判断缓冲区是否有数据未读取，有的话就不发送
//也就是超时未处理的话就不会重复通知
time.Ticker
```

## chan
```go
//固定的chan接受结构，由于是阻塞的，最好不要放到主线程执行，另外独立一个routine来执行
//没有for的话，select只会跑一遍，所以需要for
//当所有select的case都满足的话，会使用伪随机一个条件进行下去，下面的做例子就每次会不一样
for{
select {
case intChan <- 1:
case intChan <- 2:
case intChan <- 3:
}
}
```

```go
//这个是阻塞的，要等到有数据进入才会走这条语句后面的语句
<-chan
//关闭chan之后其中缓存的数据还在
close(chan)
//通常用作信号传递，是一个系统常量，多次使用并不会占用空间
struct{}{}
//单向chan和双向chan之间不支持强制转型，单向写和单向读也不支持相互之间的强制转型

//引用传递传入地址
var mapChan = make(chan map[string]*Counter,1)
countMap := nap[string[*Counter{
"count":&Counter{},
}
```

可以利用 _(chan struct{}{},并发量)_ 来控制并发量

## vs code test configuration

```json
{
            "name": "Test",
            "type": "go",
            "request": "launch",
            "mode": "test",
            "port": 2345,
            "host":"127.0.0.1",
            "program": "${workspaceFolder}/src/gogogo/loadgen/load_test.go",
            "env": {},
            "args": [],
            "showLog": true
        }
```

## sync.WaitGroup

```go
waitGrou.Add(count)
waitGroup.Done()
```
Done次数或者Add(负数)使wg等待次数小于0会引发运行时恐慌。

* Mutex : 互斥锁
* RWMutext : 读写锁 Lock/Unlock 写锁定和写解锁，RLock/RUnlock 读锁定和读解锁。Unlock会解锁因RLock而阻塞的goroutine，RUnlock只会在没有任何Lock的情况下，试图唤醒因为进行Lock而锁定的goroutine。
* sync.Cond : 条件变量



## api to be investigate

* atomic
* context


### 参考
[Go语言高级编程](
https://github.com/chai2010/advanced-go-programming-book/blob/master/ch1-basic/ch1-04-func-method-interface.md)