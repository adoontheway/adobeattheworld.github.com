---
title: Flutter Windows 桌面应用
author: ado
date: 2019-08-08 11:28:09
description: 想做一个协议测试工具
tags: [flutter]

---

# Preface

最近想做一个协议测试工具，用于功能开发阶段协议的调试。本来是想用electron来做的，突然想到之前调查flutter的时候看到了flutter可以做 PC 端native应用的schedule。于是调查了一下，发现已经可以在master分支中使用此feature了。于是就确定下来用flutter实现了。

# Reference

[Flutter for Desktop: Create and Run a Desktop Application](<https://medium.com/flutter-community/flutter-for-desktop-create-and-run-a-desktop-application-ebeb1604f1e0>)

[flutter 实战]([https://book.flutterchina.club](https://book.flutterchina.club/))

# Content

## 预备环境

上面参考的帖子里面作者是在linux环境上实现的，其实windows环境上也同样可行，只是在一些需要指定环境的地方将linux指定为windows即可。

## Steps

* 运行`flutter channel master`，然后运行`flutter upgrade`，这个操作将我本机的 **flutter** 从 **stable** 版切换到 **master** ，也就是将 *github.com/flutter/flutter* 的 **master** 分支检出到我本机。
* 然后运行 `fluter doctor`，对当前的 **flutter** 环境进行体检，一般是不会有什么问题，正常的话最后会显示 *no devices available*
* 然后需要设置 **ENABLE_FLUTTER_DESKTOP** 环境变量为 **true**，在windows上可以去环境变量里面设置这个，或者每次重新打开 CMD 的时候，设置一遍: `set ENABLE_FLUTTER_DESKTOP=true` ，PowerShell 里面的设置命令是：`$env:ENABLE_FLUTTER_DESKTOP="true"`
* 然后运行 `flutter devices` 的话，会发现已经有 **windows** 这个device 了
* 通过 git 检出官方的 **flutter desktop demo**，`git clone <https://github.com/google/flutter-desktop-embedding.git>`
* 进入到 example 下，运行 **flutter packages get**
* 成功后运行 **flutter precache --windows**
* 这个时候在其中运行 **flutter run** 会发现，ta-da，成功了

## Result

![preview](./preview.png)

这个只是一个beta功能，所以还是有一些问题的，例如，拉伸窗口的时候，窗口没有真正的windows窗口那么自然，缓冲的背景是一片黑色；需要修改native 窗口的title的话还需要手动去cc和cpp文件里面改，等等。

但是，目前来讲，能用就好。