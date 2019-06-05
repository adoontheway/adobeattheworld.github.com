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

### expected element type k_int32

MongDB 中存放的 NumberLong 对应的是 int64；

普通整型对应的是 int32；

如果使用 get_int32/int64 去取不对应的数据的话，会导致报以上错误。