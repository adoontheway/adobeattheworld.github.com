---
title: VS2017 调试 Mongocxx
author: ado
date: 2019-06-29 18:45:16
description: 在Windows上安装和调试mongocxx
tags: [mongo, c++, boost]
---

# Preface

今天遇到了个问题，需要调试mongocxx

# Prerequirement

## 安装 mongo-c-driver



* 通过 <u>git clone https://github.com/mongodb/mongo-c-driver.git</u> 克隆mongo-c-driver 源代码

* checkout 最新的**tag 1.14.0**

* 在代码的根目录下新建目录cmake-build

* 在其中运行 

  ``` sh
  cmake -G "Visual Studio 15 2017 Win64" \
    "-DCMAKE_INSTALL_PREFIX=C:\mongo-c-driver" \
    "-DCMAKE_PREFIX_PATH=C:\mongo-c-driver" \
    ..
  ```

  需要注意的是，将VS 的版版本调整到当前系统安装的 VS，不然会找不到 MSBuild.exe。

  成功的话会最后会提示以下信息:

  ```sh
  -- Compiling against Windows SSPI
  -- Configuring done
  -- Generating done
  -- Build files have been written to: F:/workspace/mongo-c-driver/cmake-build
  
  ```

* 使用 VS 打开 **cmake-build/ALL-BUILD.vcxproj**，生成

* 生成成功之后，找到 VS 的安装目录：*D:\Program Files (x86)\Microsoft Visual Studio\2017\Community\MSBuild\15.0\Bin\amd64* ，这里是**MSBuild**的目录，当然，需要注意区分*32bit/64bit*

* 将以上路径加入到环境变量Path中，这样就可以在命令行中直接使用**MSBuild**了。

*  `MSBuild.exe INSTALL.vcxproj`

* 成功后会发现系统盘根目录下多了一个目录： *mongo-c-driver*

至此，mongo-c-driver 在windows上基本安装成功。

### Refer

[Installing the MongoDB C Driver (libmongoc) and BSON library (libbson)](<http://mongoc.org/libmongoc/current/installing.html#building-windows>)

## 安装boost

* git clone https://github.com/boostorg/boost.git

* git chekout boost-1.66.0

* git submodule update --init : 这个花了好长时间

* .\bootstrap.bat

* .\b2.exe ： 这个也花了好长时间

* 安装成功后会提醒

  > The following directory should be added to compiler include paths:
  >
  >     xx\boost
  >
  > The following directory should be added to linker library paths:
  >
  >     xx\boost\stage\lib

以上步骤就说明安装成功了，只要在VS里面引入include和库路径就可以了。

### Refer

[Installing Boost](<https://github.com/boostorg/boost/wiki/Getting-Started>)

## 安装 mongocxx

windows上 mongocxx 需要依赖VS 14（2015）以及boost1.6.0：

|       | Linux    | macOS       | Windows            |
| :---- | :------- | :---------- | ------------------ |
| clang | 3.8      | 7.0 (Apple) | -                  |
| gcc   | 4.8, 5.4 | -           | -                  |
| VS    | n/a      | n/a         | 14 (2015) Update 3 |
| boost | -        | 1.5.3       | 1.6.0              |



直接编译：

```sh
cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_INSTALL_PREFIX=C:\mongo-cxx-driver -DCMAKE_PREFIX_PATH=C:\mongo-c-driver -DCMAKE_CXX_STANDARD=17 -DCMAK
E_CXX_FLAGS="/Zc:__cplusplus" -DCMAKE_BUILD_TYPE=Release ..
```

发生了错误：

> -- Auto-configuring bsoncxx to use C++17 std library polyfills since C++17 is active and user didn't specify otherwise
> CMake Error at src/bsoncxx/CMakeLists.txt:81 (find_package):
>   Could not find a configuration file for package "libbson-1.0" that is
>   compatible with requested version "1.13.0".
>
>   The following configuration files were considered but not accepted:
>
>     C:/mongo-c-driver/lib/cmake/libbson-1.0/libbson-1.0-config.cmake, version: 0.0.0

libson的版本有问题。

感觉是编译mongo-c-driver不成功，查看mongo-c-driver的目录下的 VERSION_CUIRRENT 的时候，发现其中的版本是 0.0.0 ，于是删除这个文件，删除mongo-c-dirver的cmake-build文件夹下的所有内容，重新编译的时候发现：

> -- Found PythonInterp: D:/Python27/python.exe (found version "2.7.16")
> Traceback (most recent call last):
>   File "build/calc_release_version.py", line 29, in <module>
>     from git import Git, Repo # pip install GitPython
> ImportError: No module named git
> CMake Warning at CMakeLists.txt:10 (_message):
>   BUILD_VERSION not specified and could not be calculated (script invocation
>   failed); setting library version to 0.0.0
> Call Stack (most recent call first):
>   CMakeLists.txt:97 (message)
>
> storing BUILD_VERSION 0.0.0 in file VERSION_CURRENT for later use
>   -- Using bundled libbson
> libbson version (from VERSION_CURRENT file): 0.0.0

这个在mongodb.orb 的官方论坛发现在 1.15.0 版本才修复了用 python 获取版本号的bug。

解决方案是直接将 python 生成的 VERSION_CURRENT 文件中的内容改为当前版本，也就是1.14.0，然后重新编译。

重新编译居然报错了：

> ### fatal error C1017: 无效的整数常量表达式

报错的地方是 bson-config.h, line131:

```c++
 * Define to 1 if you want extra aligned types in libbson
 */
#define BSON_EXTRA_ALIGN 
#if BSON_EXTRA_ALIGN != 1
# undef BSON_EXTRA_ALIGN
#endif
```

只需要在 `#define BSON_EXTRA_ALIGN` 后面给个值就可以了，我给了个 0 。

**ALL_BUILD** 成功之后生成一下 **INSTALL** 就可以了。

成功之后检查 *C:\mongo-c-driver\lib\cmake\libbson-1.0\libbson-1.0-config.cmake* ，发现版本号已经是1.14.0了：

```cmake
set (BSON_MAJOR_VERSION 1)
set (BSON_MINOR_VERSION 14)
set (BSON_MICRO_VERSION 0)
set (BSON_VERSION 1.14.0)
```

此时再去运行 mongocxx 的 cmake 命令，已经不会报错了。

根据cmake成功最后的提示：

> -- Generating done
> -- Build files have been written to: F:/workspace/mongo-cxx-driver/build

进入这个目录下，通过MSbuild .exe依次运行 ALL_BUILD.vcxproj和INSTALL.vcxproj；或者直接通过VS 打开ALL_BUILD.vcxproj，然后点击生成。

> Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.16.27023\include\type_traits(1271): error C2338: You've instantiated std::aligned_storage<Len, Align> with a
> n extended alignment (in other words, Align > alignof(max_align_t)). Before VS 2017 15.8, the member type would non-conformingly have an alignment of only alignof(max_align_t). VS 2017 15.8
>  was fixed to handle this correctly, but the fix inherently changes layout and breaks binary compatibility (*only* for uses of aligned_storage with extended alignments). Please define eithe
> r (1) _ENABLE_EXTENDED_ALIGNED_STORAGE to acknowledge that you understand this message and that you actually want a type with an extended alignment, or (2) _DISABLE_EXTENDED_ALIGNED_STORAGE
>  to silence this message and get the old non-conformant behavior. [workspace\mongo-cxx-driver\build\src\bsoncxx\bsoncxx.vcxproj]

我的VS 版本是 15.9.7 ，所以是可以用 `_ENABLE_EXTENDED_ALIGNED_STORAGE`  的。

在CMakeLists.txt 中加入:

```cmake
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D _ENABLE_EXTENDED_ALIGNED_STORAGE")
```

接着All_Build，还在报错。。。

算了，Windows真不友好.....

### Refer

[Installing the mongocxx driver](<http://mongocxx.org/mongocxx-v3/installation/>)

[mongo-c-driver-1.14.0: wrong version in pkg-config files](<https://jira.mongodb.org/browse/CDRIVER-3022>)