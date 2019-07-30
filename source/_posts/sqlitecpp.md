---
title: SQLite的使用
author: ado
date: 2019-07-23 09:47:16
description: c++中使用sqlite
tags: [cpp,sqlite]

---

# Preface 

需要使用Sqlite，所以研究一下

# Reference

[sqlite docs](<https://www.sqlite.org/docs.html>)

上面文档里面基本都有，安装，api的使用。

# Steps

## Install

[download](<https://www.sqlite.org/download.html>) 从这个页面下载安装报，我选择的是 sqlite-autoconf-3290000.tar,gz。

解压之后依次运行：

```sh
./configure
make
```

就可以安装成功。

## Usecase

此时可以在项目中使用了。

```c++
#include  "sqlite3.h"

```

