---
title: WMWare上CentOS遇到的问题
author: ado
date: 2019-05-10 11:32:41
description: 在VMWare上安装CentOS遇到的问题记录
tags: centos

---

# Preface

在 VMWare 上安装 CentOS 遇到的的一些问题记录



### Name or service not known

> vim /etc/resolv.conf

```sh
# 二选一
# Google提供的DNS
nameserver 8.8.8.8
nameserver 8.8.4.4
# 国内的DNS
nameserver 114.114.114.114
nameserver 114.114.114.115
```

> sudo service network restart

