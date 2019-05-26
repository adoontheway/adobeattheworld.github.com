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
* [MongoDB Manual](https://docs.mongodb.com/manual/)
* ObjectID :  0-3 时间戳 4-6 机器标识码 7-8 进程id 9-12 随机数，单位：字节![objid](https://adobeattheworld.github.io/images/objidtime.png)
* **find** 语法 `db.users.find({},{"username":1,"email":0})` 只需要返回 *username* 不需要返回 *email* 。
* batchInsert 已经废弃了，直接使用 insert 传入数组可以插入多条数据

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

* $ne : not equal !=
* $addToSet : 保证添加元素的唯一
* $push :  配合 $each 和 $slice用于对数组添加元素
* $each : 对每个元素进行操作
* $slice :  必须是负整数，保证数组长度不大于，可配合 $sort 使用
* $pop : 删除一个元素， {"$pop":{"key":1}}从末尾删除一个元素，-1则从头部删除一个元素
* $pull :  删除特定元素
* $inc : 自加，可以接受负数
* $set : 设值
* $setOnInsert : 字段只会在文档插入的时候更新，之后所有更新操作这个字段都不会改变。 `db.users.update({},{"$setOnInsert":{"createdAt":newDate}},true)`
* $lt : less than  <
* $lte : less than or equal <=
* $gt : greater than >
* $gte : greater than or equal >=
* $in : 查询一个键的多个值 `db.raffle.find({"ticket_no":{"$in":[1,2,3,4]}})`
* $nin: not in
* $and : 
* $nor : 
* $or : 在多个键中查询任意的给定值 `db.raffle.find("$or": [{"ticket_no":555}, {"winner":true}])` 或者可以和 **$in**组合使用：`db.raffle.find({"$or" : [{"ticket_no":{"$in":[555,.666,777]}},{"winner":true}]})`
* $not :  对下面取反只要用这个替换就可以了 `{"$not":{"$mod":[5,1]}}`， $not 可以与正则表达式联用。
* $mod : 取模运算 `db.users.find({"id_num":{"$mod":[5,1]}})`查询 **id_num** <u>对5取模余数为1</u> 的文档。
* $exists : `db.c.find({"z":{"$in":[null],"$exists":true}})` 匹配键值为null，且存在的元素
* $size : 匹配数组尺寸 `db.food.find({"fruit": {"$size":3}})`
* $elemMatch : 匹配数组元素
* $where :  终极查询语句，<u>应该严格限制或者消除 $where 的使用。禁止任何终端用户使用任意的 $where 语句</u>。并且查询速度也很慢。
* $maxscan : 指定本次查询扫描文档数量的上限。
* $min :  查询的开始条件。 再这样的查询中，文档必须与索引的键完全匹配，再内部使用时，通常应该使用 $gt 代替 $min 。可以使用 $min 强制指定一次索引扫描的下边界。
* $max : 查询的结束条件。与上面相反。
* $showDiskLoc : 再查询结果中添加这个用于显示该条结果再磁盘上的位置。 `db.foo.find()._addSpecial('$showDickloc',true)`，**cursor** 有对应的方法 [showDiskLoc()](<https://docs.mongodb.com/manual/reference/operator/meta/showDiskLoc/index.html>)。

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

## 8. findAndModify

getLastError 可以返回上一次更新的有关信息：

```javascript
db.runCommand({getLastError:1})
```

如果有多个线程在查询和更新同一个文档的话，可以用 findAndModify 来避免这类问题，他会返回一个匹配结果并且进行更新，然后返回修改之前的结果。

这个接口很重要，还可以用来安全的删除和新建。

> 可用指令： query, sort,update,remove,new,fields,upsert
>
> 其中，update 和 remove 必须有且只能有一个，否则会报错。



## 9. $slice

`db.blog.posts.findOne(criteria, {"comments":{$slice : -10 }})`

找到匹配的数据并且选取 **comments** 里面倒数 **10** 条数据。

`db.blog.posts.findOne(criteria, {"comments":{$slice:[3,5]}})`

找到匹配项并且选取 **comments** 里面索引 【4】 开始的 【5】 个数据。

## 10. 启动参数

启动加入 *--noscripting* 参数可以完全关闭服务端 **JavaScript** 的执行。

## 11. 游标

```javascript
var cursor = db.test.find();
//use case 1
cursor.forEach( function(x) {...})
//use case 2
while(cursor.hasNext()) {}
```

调用 find 时，shell不会立即查询数据库，二十等待真正开始要求获取结果的时候才发送查询，这样在执行之前可以给查询附加额外的选项。几乎游标对象的每个方法都返回游标本身，这样就可以按任意顺序组成方法链。例如，一下三个是等价的：

```javascript
var cursor = db.foo.find().sort({x:1}).limit(1).skip(10);
var cursor = db.foo.find().limit(1).sort({x:1}).skip(10);
var cursor = db.foo.find().skip(10).sort({x:1}).limit(1);
```

*hasNext* 会有**4MB**数据的限制，如果请求的下一份数据超过 **4MB** 那么只会返回 **4MB** 的数据。

## 12. 比较顺序

如果一个键可能有多个类型的值，默认的排序顺序由小到大：

1. 最小值
2. null
3. 数字：整型，长整形，双精度
4. 字符串
5. 对象/文档
6. 数组
7. 二进制数据
8. 对象ID
9. 布尔型
10. 日期型
11. 时间戳
12. 正则
13. 最大值

## 13. Skip 略过大量结果

skip 略过过多的话会变得很慢，因为要找到需要被略过的数据，然后再抛弃这些数据。所以，尽量不要使用skip对结果分页。

可以根据具体逻辑来保存上一个查询最后的一个结果来获取下一页。

## 14. 随机选取文档的技巧

可以预先给文档设计一个 random 值，然后查询的时候随机一个数并根据这个数查询就可以了。

## 15. 快照 snapshot

对整个文档进行查询和处理的时候，由于文件大小变更，可能导致文档进行了移位，从而导致多次处理同一个文档，这个时候需要使用快照来处理。

```javascript
db.foo.find().snapshot()
```

**mongodump** 默认是在快照上使用查询。

当然，快照会使查询变慢。

## 16. 命令

* db.rumCommand
* db.adminCommand

命令名称必须是命令中的第一个字段。



