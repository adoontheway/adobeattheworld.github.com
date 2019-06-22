---
title: 一些问题
author: ado
date: 2019-06-20 17:04:34
description: 今天遇到的一些问题备份
tags: [git]

---

# Preface

基于自己的本地镜像制作新人入职用的镜像的时候，遇到一些问题，特此记录。

## Linux上清理git信息的命令

git help看起来好晦涩，搜索中文又是一片水，最后搜索到如下信息：

```sh
git config --global --unset credential.helper
```

[Linux - How to Remove Git Credentials](https://stackoverflow.com/questions/44246876/linux-how-to-remove-git-credentials)

## Linux 清理svn信息

去svn的根目录下执行：

```sh
find -type d -name '.svn' -exec rm -rfv {} \;
```

 [How to recursively remove .svn metadata folders?](https://askubuntu.com/questions/35917/how-to-recursively-remove-svn-metadata-folders)