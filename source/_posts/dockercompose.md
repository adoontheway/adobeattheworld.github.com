---
title: Docker Compose安装
author: ado
date: 2019-05-10 14:37:55
description: 安装Docker Compose并用他来建立一个Goland注册服务器
tags: [docker]
---



# Preface

嗯，Goland过期了。我记得我下载的Community版本呢，为何？

于是在Github上找到个Goland的注册服务器的 Docker 镜像，但是需要安装 Docker Compose。

虽然知道这样是不对，但是有点穷，用了VS Code写Golang之后就用不回去LiteIDE了；用了Goland之后用不回VS Code了。

有能力的话我还是会支持正版的，毕竟我也是偶尔扶老太太过马路的啥的。

# Steps

## 安装Compose

[Compose官方安装参考文档](https://docs.docker.com/compose/install/)

官方提醒：

> For `alpine`, the following dependency packages are needed: `py-pip`, `python-dev`, `libffi-dev`, `openssl-dev`, `gcc`, `libc-dev`, and `make`

linux alpine 版本需要安装以上依赖项，还好我是 centos，直接运行下面的命令就可以了：

```sh
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

修改 docker compose 执行文件的权限：

```sh
sudo chmod +x /usr/local/bin/docker-compose
```

运行 docker-compose version，输出如下：

```sh
docker-compose version 1.24.0, build 0aa59064
docker-py version: 3.7.2
CPython version: 3.6.8
OpenSSL version: OpenSSL 1.1.0j  20 Nov 2018
```

说明安装成功了。

## 安装JetBrains注册服务器

从此处 [crazy-max/docker-jetbrains-license-server](<https://github.com/crazy-max/docker-jetbrains-license-server>) 克隆源代码到本地，并修改此文件夹的权限。

> 继续昨天的事情，咸鱼

## docker-compose: command not found

直接运行docker-compose没有问题，加上sudo就报这个错误了。

解决办法：

```sh
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

参考 ： [sudo: docker-compose: command not found](https://stackoverflow.com/questions/38775954/sudo-docker-compose-command-not-found)

还有另一种解决办法是切换用户权限，可惜我没有管理员密码。



接着创建文件夹 /var/jls，

接下来将clone下来的 docker-jetbrains-license-server 下的example/compose下的文件复制到 /var/jls 中，

然后再文件夹中运行如下命令：

```sh
touch acme.json
chmod 600 acme.json
docker-compose up -d
docker-compose logs -f
```

然后运行：

```sh
sudo docker-compose up -d
```

完成



# 结论

鼓捣了半天，发现这个居然不是一个注册服务器，只是一个FLS，即[Floating License Server](https://www.jetbrains.com/help/license_server/getting_started.html)

至此，创建Goland激活服务器失败，用回 VS Code了。