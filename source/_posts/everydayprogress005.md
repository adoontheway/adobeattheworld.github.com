---
title: 每天进步一点点005-mongocxx嵌套数组中插入文档
author: ado
date: 2019-12-07 16:28:59
description: 使用mongocxx/bsoncxx的时候遇到的在数组中嵌入文档的写法比较晦涩，所以备忘于此
tags: [c++,mongo]
---

# Preface
这两天在写一个逻辑，需要向mongodb中插入一个对象，这个对象还有一个成员是一个数组，需要往这个数组中插入一些文档。  
在此记录的原因是这里我老是会报错，报错如下：  
> Error Mongodb writedata error: expected a key but found none

嗯，反复的弄了几遍都没有结果，在这里来求助官网了。

# Demo

## Demo0 
```c++
#include <iostream>
#include <bsoncxx/json.hpp>
#include <bsoncxx/builder/stream/array.hpp>
#include <bsoncxx/builder/stream/document.hpp>
#include <bsoncxx/builder/stream/helpers.hpp>
#include <bsoncxx/types.hpp>

using bsoncxx::builder::stream::close_array;
using bsoncxx::builder::stream::close_document;
using bsoncxx::builder::stream::document;
using bsoncxx::builder::stream::finalize;
using bsoncxx::builder::stream::open_array;
using bsoncxx::builder::stream::open_document;

using namespace bsoncxx::types;
using namespace std;

int main()
{
    auto doc = document{};
    auto arr = bsoncxx::builder::stream::array{};
    doc << "myKey" << "myValue";
    doc << "foo" << b_bool{false} << "baz" << b_int32{1234} << "quz" << b_double{1.24};
    arr << 1 << 2 << b_bool{true};
    doc << "mySubDoc" << open_document << "subdoc key" << "subdoc value" << close_document;
    doc << "mySubArr" << open_array << 1 << b_bool{false} << "hello" << close_array;
    auto doc1 = doc << "tasks" << open_array;
    for(auto i : {1,2,3})
    {
        doc1 << open_document << "a" << "b" << close_document;
    }
    doc1 << close_array;
    auto myQuery = document{} << "foo" << "bar" << finalize;
    doc << bsoncxx::builder::concatenate(myQuery.view());
    cout << bsoncxx::to_json(doc.view()) << endl;
    return 0;
}
```
以上输出结果如下：
```json
{ 
    "myKey" : "myValue", 
    "foo" : false, 
    "baz" : 1234, 
    "quz" : 1.2399999999999999911, 
    "mySubDoc" : { "subdoc key" : "subdoc value" }, 
    "mySubArr" : [ 1, false, "hello" ], 
    "tasks" : [ 
        { "a" : "b" }, 
        { "a" : "b" }, 
        { "a" : "b" } 
        ], 
    "foo" : "bar" 
}

```

## Demo1

```c++
#include <iostream>
#include <bsoncxx/json.hpp>
#include <bsoncxx/builder/stream/array.hpp>
#include <bsoncxx/builder/stream/document.hpp>
#include <bsoncxx/builder/stream/helpers.hpp>
#include <bsoncxx/types.hpp>

using bsoncxx::builder::stream::close_array;
using bsoncxx::builder::stream::close_document;
using bsoncxx::builder::stream::document;
using bsoncxx::builder::stream::finalize;
using bsoncxx::builder::stream::open_array;
using bsoncxx::builder::stream::open_document;

using namespace bsoncxx::types;
using namespace std;

int main()
{
    bsoncxx::builder::stream::document builder{};
    builder << "a" << 1;
    auto insert_value = builder << "list1" << open_array;
    for(auto i : {0,0,0})
    {
        auto insert_value_1 = insert_value << bsoncxx::builder::stream::open_document
                << "b" << 1
                 << "c" << 1
                 << "d" << 1
                 << "e" << 1
                 << "list2" << open_array;
        for(auto j : {0,0})
        {
            insert_value_1  << bsoncxx::builder::stream::open_document
                            << "c" << 1
                            << "d" << 1
                            << bsoncxx::builder::stream::close_document;
        }
        insert_value_1 << close_array << close_document;
    }
    insert_value << close_array;
    cout << bsoncxx::to_json(builder.view()) << endl;
    return 0;
}
```
结果：  
```json
{ 
    "a" : 1, 
    "list1" : [ 
        { 
            "b" : 1, 
            "c" : 1, 
            "d" : 1, 
            "e" : 1, 
            "list2" : [ 
                { "c" : 1, "d" : 1 }, 
                { "c" : 1, "d" : 1 } 
            ] 
        }, 
        { 
            "b" : 1, 
            "c" : 1, 
            "d" : 1, 
            "e" : 1, 
            "list2" : [ 
                { "c" : 1, "d" : 1 }, 
                { "c" : 1, "d" : 1 } 
            ] 
        }, 
        { 
            "b" : 1, 
            "c" : 1, 
            "d" : 1, 
            "e" : 1, 
            "list2" : [ 
                { "c" : 1, "d" : 1 }, 
                { "c" : 1, "d" : 1 } 
            ] 
        } 
    ] 
}
```

# Reference
* [bsoncxx/builder_stream](https://github.com/mongodb/mongo-cxx-driver/blob/master/examples/bsoncxx/builder_stream.cpp)
* [Linux与Windows共享文件夹之samba的安装与使用（Ubuntu为例）](https://www.cnblogs.com/gzdaijie/p/5194033.html)