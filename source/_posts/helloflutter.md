---
title: Hello, Flutter
author: ado
date: 2019-06-26 14:26:53
description: flutter官方demo入门
tags: [flutter]
---

# Preface

最近一些工具登陆不上去了，忘记了一些密码。

想想是不是需要写一个生成和记住密码的 app 自用一下。

想起了去年安装了一下但是由于模拟器的原因无法跑起来的 flutter，就打算用一下。

# Reference

## 第三方包的使用

[pub.dev](<https://pub.dev/flutter>) : 这里是找 **Flutter** 第三方包的地方

找到需要的第三方包之后，就可以去项目的 *pubspec.yaml*的 <u>dependencies</u>或者 <u>dev_dependencies</u> 下按照格式填好报名和版本，然后保存，**vs code**会自动运行如下命令来下载添加的依赖包：

```sh
flutter package get
```

这个感觉和 **npm** 很像。

# Content

入门课很简单，教你如何使用三方包以及 *ListView* 的使用。

照着官方敲代码，也就几十行。



# Prototype

打算在手机上利用此app新建和存储密码；

本地存储方式是利用sqlite；

数据加下密；

最好能有个远程云盘备份的啥的。

原型图在此：

![prototype](<https://adobeattheworld.github.io/images/helloflutter.png>)

