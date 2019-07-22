---
title: MongoDB 使用过程中遇到的问题
author: ado
date: 2019-06-05 10:15:44
description: MongoDB使用过程中遇到的问题，不限语言
tags: [mongo]

---

# Preface

Reserved

# Content

## MongoDB C++ 驱动查询参数遇到的问题

> bsoncxx/builder/stream/value_context.hpp:69: error: call of overloaded ‘append(unsigned int&)’ is ambiguous
>          _core->append(std::forward<T>(t));

这是MongoDB C++ driver：mongocxx 设计上是一个问题，int类型只支持int32和int64类型，而我们使用了unsigned int的一个别名，所以导致了报错，转型int就可以了。

```c++
using bsoncxx::builder::stream::document;
using bsoncxx::builder::stream::finalize;

bsoncxx::document::value query_value = document{} << "a" << (int)a << "b" << (int)b << finalize;
```

### 参考

[bsoncxx::builder::core::append of unsigned int](<https://jira.mongodb.org/browse/CXX-617>)

## expected element type k_int32

MongDB 中存放的 NumberLong 对应的是 int64；

普通整型对应的是 int32；

如果使用 get_int32/int64 去取不对应的数据的话，会导致报以上错误。

## mongorestore

```sh
mongorestore /h 192.168.2.160 /port 27017 /dir currentdir -d targetdatabase /bypassDocumentValidation
```

这个是Windows上面的操作，具体的可以参考:

```sh
mongorestore --help
```

## mongoshell 执行js脚本

```sh
mongo 192.168.0.200/test e:/test.js
```

这个是不带验证的，验证的要另加。

以下是一段数据整理的脚本，数据量多的话可以利用 **cursor** 的分页功能:

```js
var s = db.getMongo().startSession()
s.startTransaction()

try {
	var cursor = db.coll.find({});
	var total = cursor.count()
	var operated = 0;
	
	function each(x){
		if(x.time > ISODate("2019-05-05T05:00:00.00+00:00"))//如果这个字段大于这个时间
		{
			print(x.time)
			db.coll.update({_id:x._id}, {"$set":{"a":a,b:c}},true)
		}
		operated++;
		print((operated*100/total).toFixed(2)+"%")
	} 
	cursor.forEach(each);
	s.commitTransaction();
}catch(e){
	print(e);
	s.abortTransaction();
}
print("Finished")
```

以下是建库建集合的脚本：

```js
var conn = new Mongo('192.168.0.200:27017');
var db = conn.getDB('newdb');
db.createCollection('new_coll');
db.game_replay.ensureIndex({hehe:1});
db.game_replay.insertOne({hehe:'test0000000000000'});
print('Please clean test data in newbd.new_coll and grant user to write/read this database');
```

