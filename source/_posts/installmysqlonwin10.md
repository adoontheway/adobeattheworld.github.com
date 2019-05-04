---
title: Win10上安装Mysql
layout: post
tags: mysql
---
# 前言
公司项目管理用的svn，一开就建议公司用git，但是似乎发言权不够。
连项目管理和bug追踪都不做的时候我推荐用禅道或者jira也是勉勉强强的接受了禅道的。
心累。
最近些的子项目刚过了第一波测试，我本地又加了很多新功能，自测都不好开展，于是就打算在本地安装git环境做好代码管理。
由于本机是windows10家庭版，且装有vbox和vmware，开发是在vmware上装centos，centos上又安装了qt creator。
总之是蛮卡的，加之之前用vbox+ubuntu做开发，经常被一些基于vbox的模拟器搞崩。
所以，实在不敢动虚拟机中的开发环境，所以决定在windows上装个git。
在gitlab和gogs之间权衡了一下，windows10家庭版没有hyper-v，所以选择了gogs。
gogs支持mysql，postgresql，mssql还有sqlite，最近正在撸自己的服务端框架，所以庄个mysql吧。
反正是临时的，后面还是要去找管事的大爷们安利一下git。

### 下载mysql
官网下载mysql的zip安装包解压下来居然是个完整的mysql程序目录。
懵逼~~~
之前不都是安装程序么。。。

### 创建配置文件

在解压后的mysql根目录下创建一个文件：*my.ini*
```ini
[mysqld]
# basedir mysql的安装路径，也就是解压路径
basedir=D:\\installers\\mysql-8.0.15-winx64
# 数据存放路径
datadir=D:\\installers\\mysql-8.0.15-winx64\\data
```
注意下windows的路径需要双斜杠的。
压缩包里面默认是没有data文件夹的，需要自己建一个。当然，你也可以把数据文件夹建到别的地方去，只要这里配置一下就可以了（我理所当然的想的）。

### 数据文件夹的初始化
调用 *mysqld* 的 *--initialize* 或者 *--initialize-insecure* 来初始化数据文件夹。区别是 *--initialize* 默认会为 *root* 生成一个随机密码。
运行：
```sh
> mysql-8.0.15-winx64>mysqld --defaults-file=./my.ini --initialize --console

2019-04-22T04:23:08.321323Z 0 [System] [MY-013169] [Server]mysql-8.0.15-winx64\bin\mysqld.exe (mysqld 8.0.15) initializing of server in progress as process 16292
2019-04-22T04:23:11.783989Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: sqaOhgP)e2y9
2019-04-22T04:23:12.855674Z 0 [System] [MY-013170] [Server] mysql-8.0.15-winx64\bin\mysqld.exe (mysqld 8.0.15) initializing of server has completed
```
运行完成之后会发现 *data* 文件夹里面生成了很多东西。
**千万要记得 --defaults-file 名字不能输错，顺序要在最前面 **

### 启动mysql
```
> mysqld --console
```

### 连接mysql
```
> mysql -u root -p
```

### 改密码
```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '12345';
```

### 加用户
```
mysql> CREATE USER 'root'@'127.0.0.1' IDENTIFIED BY '12345';
```

### 参考
[Gogs在windows下安装](
https://gogs.io/docs/installation)
[在Windows安装Mysql8.0](
https://dev.mysql.com/doc/refman/8.0/en/windows-install-archive.html)