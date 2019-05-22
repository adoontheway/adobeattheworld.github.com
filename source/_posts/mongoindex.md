---
title: mongo相关
author: ado
date: 2019-05-16 20:41:16
description: mongodb 和 MongoDB权威指南 的相关笔记
tags: [mongo]
---



# Preface

这里是 **mongodb** 日常和读 **MongoDB权威指南** 记笔记的地方

# Content

## 1. 基本知识

* [MongoDB 教程](http://www.runoob.com/mongodb/mongodb-tutorial.html)
* ObjectID :  0-3 时间戳 4-6 机器标识码 7-8 进程id 9-12 随机数，单位：字节![objid](https://adobeattheworld.github.io/images/objidtime.png)

## 2. 性能优化

### 2.0 索引

**mongodb** 建索引的方法:

```js
db.collection_name.createIndex({'key0':1,'key1':1,'key2':-1})
```

以上是建联合索引，当然也可以单独给每个字段建索引

参考 ：

[mongodb 索引优化](https://www.jianshu.com/p/a0e3e18ace77)

> db.collection_name.drop() 之后需要重新建索引，
>
> 目前是没有办法在创建集合的时候一起创建索引的。

## 3. 启动脚本

创建一个 **.mongorc.js** ，可以在其中加入一些内容，例如，添加全局变量，为太场的名字创建简短的别名。最常见的用途是移除那些比较危险的**shell**辅助函数，例如*dropDatabase* 或者 *deleteIndexes*:

```javascript
var no = functio () {
    print("Not on my watch.");
};
//禁用数据库删除
db.dropDatabase = DB.prototype.dropDatabase = no;
//禁用集合删除
DBCollection.prototype.drop = no;
//禁用索引删除
DBCollection.prototype.dropIndex = no;
```

启动 **shell** 时可以指定 *--norc* 参数来禁止加载 **.mongorc.js** 的加载。

## 4. can't save a DBQuery object

*db.coll.find({})* 查询出来的结果是一组数据，直接用于更新或者插入操作可能会出现上面的报错，要使用的话需要在后面加个索引，当然你还是需要判断里面有没有数据，或者使用 *db.coll.findOne({})* 。

## 5. 条件

* $ne : not equal ?
* $addToSet : 保证添加元素的唯一
* $push :  配合 $each 和 $slice用于对数组添加元素
* $each : 对每个元素进行操作
* $slice :  必须是负整数，保证数组长度不大于，可配合 $sort 使用
* $pop : 删除一个元素， {"$pop":{"key":1}}从末尾删除一个元素，-1则从头部删除一个元素
* $pull :  删除特定元素
* $inc : 自加，可以接受负数
* $set : 设值
* $setOnInsert : 字段只会在文档插入的时候更新，之后所有更新操作这个字段都不会改变。 `db.users.update({},{"$setOnInsert":{"createdAt":newDate}},true)`

## 6. 文档移动

改变某些属性值会导致文档移动，例如：数组添加元素，字符串设置为更长的值得。某些是不会导致文档移动的，例如：对一个数字类型的元素进行运算设值。

集合预留的填充因子 **padding factor** 可以通过 *db.coll.stats()* 查看。

设置 usePowerOf2Sizes 为 true 可以让分配的空间得到的块空间大小都是2的幂。

```javascript
db.runCommand("collMod":collectionName, "usePowerOf2Sizes":true)
```

## 7. Upsert

**upsert**设置为*true*可以免去**query**查询是否存在，不存在的话会创建一个：

```javascript
db.collectionname.update(query,update, upsert);
```

这样具备原子性且效率更高。

 











