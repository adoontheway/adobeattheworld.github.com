---
title: 在Flutter中使用Protobuf
author: ado
date: 2019-08-09 09:41:04
description: Flutter中使用protobuf的过程与遇到的问题备忘
tags: [dart,protobuf,websocket]

---

# Preface

上一篇中提到的工具，需要使用自定义字节流协议，自定义字节流中有使用到protobuf。

# Reference

* Dart 官方protoc插件：[Dart protoc-plugin](<https://pub.dev/packages/protoc_plugin#-installing-tab->)
* Protobuffer 官方Dart手册 [Protocol Buffer Basics: Dart](<https://developers.google.com/protocol-buffers/docs/darttutorial>)

# Content

## 添加protobuf支持

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

之后就可以直接在 flutter 上导入使用了

## 添加websocket支持

在]dart.dev](<https://pub.dev/packages/web_socket_channel#-installing-tab->)上搜索到websocket 包，然后再pubspec.yaml中导入进来。使用方式：

```dart
import 'package:web_socket_channel/status.dart' as status;
import 'package:web_socket_channel/io.dart';
IOWebSocketChannel _channel;
_channel = await IOWebSocketChannel.connect(addr);
_channel.sink.add('connected.');
_channel.stream.listen((message){

});
```

但是我的客户端连接服务端websocket服务的时候，握手就报错了：

>  WebSocketChannelException: WebSocketException: Bad response
> 'Sec-WebSocket-Accept' header

所以需要再vs code中调试flutter。

## vs code中调试flutter

本来以为 [Dart-Code 插件](<https://github.com/Dart-Code/Dart-Code>) (也就是 Dart 的 vs code插件)不支持flutter desktop 调试的。

> Dart-code 插件负责 Dart 语言的调试，提醒和高亮
>
> flutter 插件只负责 flutter 包提醒，以及 device获取等辅助功能，并没有直接调试 flutter项目的功能。

于是找到了 Dart-code 在github上的开源地址，打算修改一下让他支持 flutter desktop 调试，因为之前写过 vs code 插件，所以改这个应该难度是不大的。

但是今早来不小心搜索到了 flutter 的 maillist中的这个帖子，[VSCode Running Desktop App with Play button](<https://groups.google.com/forum/#!topic/flutter-desktop-embedding-dev/nxU9bsAMS3I>)，发现 Dart-code 是支持 flutter desktop 调试的，只需要在 launch.json 的 configuration 中添加如下配置即可：

```json
{
    "name": "Flutter Desktop",
    "type": "dart",
    "request": "launch",
    "deviceId": "windows",
    "flutterMode": "debug",
    "env" : {
        "ENABLE_FLUTTER_DESKTOP":true
    }
},
```

这样就可以直接调试 flutter desktop 了。