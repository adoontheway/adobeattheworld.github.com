---
title: MongoDB状态监视工具
author: ado
date: 2019-06-12 20:28:01
description: MongoDB状态监视相关
tags: [mongo]
---

# Preface

今天在研究MongDB分片集群是时候，突然发现连接到MongoDB的时候出现了这个提示：

> MongoDB shell version v4.0.10
> 
>
> Enable MongoDB's free cloud-based monitoring service, which will then receive and display
> metrics about your deployment (disk utilization, CPU, operation statistics, etc).
>
> The monitoring data will be available on a MongoDB website with a unique URL accessible to you
> and anyone you share the URL with. MongoDB may use this information to make product
> improvements and to suggest MongoDB products and deployment options to you.
>
> To enable free monitoring, run the following command: db.enableFreeMonitoring()
>
> To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
> ---

重点在这里：

> 运行 db.enableFreeMonitoring() 可以激活免费的监视器

# Content

## db.enableFreeMonitoring 

于是本着研究与好奇的心里，运行了一下：

```sh
db.enableFreeMonitoring() 
```

结果输出：

```json
{
     "state" : "enabled",
     "message" : "To see your monitoring data, navigate to the unique URL below. Anyone you share the URL with will also be able to view this page. You can disable monitoring at any time by running db.disableFreeMonitoring().",
     "url" : "https://cloud.mongodb.com/freemonitoring/cluster/xxxxxxxx",
     "userReminder" : "",
     "ok" : 1
}
```



然后，将这里的url用浏览器打开，就可以依次看到：

* Operation Execution Times : 操作执行次数，包括，读，写，命令三种操作的执行次数
* Disk Utilization :  磁盘的利用率，包括所有磁盘的最大利用率，所有磁盘的平均利用率
* Documents : 文档每秒的平均（查询返回，插入，更新，删除）率
* Memory ： 常驻内存，MongoDB的虚拟内存，MongoDB的映射（Mapped）内存,映射内存基本上与数据库的占用尺寸接近。 WiredTiger不会用内存来映射文件，所以这里就会是0.
* Network-Input/Output：流入字节数与流出字节数
* Opcounters ：操作计数器，采样起见平均每秒的（插入，查询，更新，删除，获取更多GetMore,命令）的执行率
* Opcounters-Replication ：操作计数器复制
* Query Targeting：查询目标
  * Scanned/Returned：
  * 扫描对象/返回：
* Queues：
  * Readers 等待读锁定的已入队操作数
  * Writers 等待写锁定的已入队操作数
  * Total  所有等待锁的已入队操作数
* System Cpu Usage：也就是系统的cup使用状态

## Free Monitoring

MongoDB的[Free Monitoring](<https://docs.mongodb.com/manual/reference/method/js-free-monitoring/>) api主要有三个。

* db.disableFreeMonitoring() 禁用
* db.enableFreeMonitoring() 激活
* db.getFreeMonitoringStatus() 状态

如上，都很好理解。

小小的调查了一下，似乎并不支持将Free Monitoring的结果输出到指定的页面。

## 7个工具

[6个有用的MongDB性能监视工具](<https://www.tecmint.com/monitor-mongodb-performance/>)

### Mongostat

### Mongotop

### serverStatus

也就是

```sh
db.serverStatus()
```

通过搜集统计当前Mongo实例，返回数据库的状态概览。

上面调查Free Monotoring是否可以修改输出地址的时候，导向的 metrics 相关属性就看到了。

### dbStats

```sh
db.stats()
```

返回的是指定数据库的存储统计信息

### collStats

```sh
db.runCommand({collStats:"coll", scale:1024})
```

与 dbStats 类似，只不过级别是集合了，输出额外的包括了集合里的对象书，集合的尺寸，集合消耗的硬盘空间，以及他相关的索引信息。

### replSetGetStatus

```sh
db.adminCommand({replSetGetStatus:1})
```



