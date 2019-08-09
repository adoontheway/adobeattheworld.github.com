---
title: 在Flutter中使用Protobuf
author: ado
date: 2019-08-09 09:41:04
description: Flutter中使用protobuf的过程与遇到的问题备忘
tags: [dart,protobuf]

---

# Preface

上一篇中提到的工具，需要使用自定义字节流协议，自定义字节流中有使用到protobuf。

# Reference

* Dart 官方protoc插件：[Dart protoc-plugin](<https://pub.dev/packages/protoc_plugin#-installing-tab->)
* Protobuffer 官方Dart手册 [Protocol Buffer Basics: Dart](<https://developers.google.com/protocol-buffers/docs/darttutorial>)

# Content



按照Protobuffer官方手册，直接运行：

```sh
protoc -I=$SRC_DIR --dart_out=$DST_DIR $SRC_DIR/addressbook.proto
```

会报错

>'protoc-gen-dart' 不是内部或外部命令，也不是可运行的程序
>或批处理文件。
>--dart_out: protoc-gen-dart: Plugin failed with status code 1.

这是因为没有安装 protoc dart插件所致。

dart protoc插件的描述信息里面写道，全面只是proto2，但是要求protobuf的版本>= 3.0.0，我本机的 protobuf 版本是3.7.1。

按照官方插件的安装指引，在命令行中运行：

```sh
pub global activate protoc_plugin
```

成功之后就可以用上面的命令将 proto 生成 dart 文件。

如果想要在flutter项目中使用的话，直接在pubspec.yaml的dependencies中添加：protoc_plugin: ^17.0.5，在vs code中保存会直接触发flutter package get，或者自己去项目的目录下手动运行这个命令也可以。