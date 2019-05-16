---
title: cannot find main module
author: ado
date: 2019-05-15 11:51:32
description: cannot find main module; see 'go help modules'
tags: [go]
---

# Content

昨天新建了个项目，在 **main.go**  里面 *package main* 居然报错了，使用 *go build* 的时候也是报错，报错信息如下:

```sh
go: cannot find main module; see 'go help modules'
```

网上搜索也有很多这个问题，发现基本都是 **go mod** 的问题。

会想起上周在系统里激活了 **go mod** 的相关配置，也许是这个问题吧。

于是进入报错的目录中打开命令行：

```sh
go mod init
```

报错信息不见了，**go build** 也正常运行。

然后：

```sh
go mod tidy
```

莫非激活了 **go module** 配置就必须要强行使用？

不管了，收工。