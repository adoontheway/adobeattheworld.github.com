---
title: Docker搭建mysql和Redis集群
date: 2019-05-04 14:25:12
tags: [docker, redis]
description: 在centos上搭建mysql和redis集群遇到的一些问题记录
---





### 查看docker mysql镜像信息

查看mysql镜像的ip

```shell
docker exec chai_mysql cat /etc/hosts
```

进入mysql镜像命令行

```shel
docker exec -it chai_mysql bash
```

docker run my_mysql找不到container的问题，要用start命令

```shell
docker start my_mysql
```

## 问题

### 1. redis容器中的redis.conf访问被拒

```shell
sudo docker run -it -v /home/Me/docker/chai/redis-master.conf:/usr/local/etc/redis/redis.conf --name redis-master redis /bin/bash
  
redis-server /usr/local/etc/redis/redis.conf
```
### 2. Fatal error, can't open config file '/usr/local/etc/redis/redis.conf

```shell
sudo docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
ac746d069659        redis               "docker-entrypoint..."   8 minutes ago       Up 6 minutes        6379/tcp                            redis-master

```

```shell
docker logs redis-master
systemctl status -l docker.service
```

![Docker Redis cluster](../images/dockerredis-0.png)

### 3. SELinux is preventing /usr/local/bin/redis-server from read access on the file redis-master.conf
都是**SELinux**搞的鬼，编辑*/etc/sysconfig/selinux* 将**selinux**设置为**disabled**之后重启就可以了。

### 4. error creating overlay mount to /var/lib/docker/overlay2/

[参考](https://colobu.com/2018/06/28/Error-response-from-daemon-error-creating-overlay-mount-to-var-lib-docker-overlay2/)

## 重新建立mysql和docker容器

上面的解决方案中清理掉了所有的docker镜像和容器
```shell
docker pull mysql
sudo docker run --name chai_mysql -p 3306:3306 MYSQL_ROOT_PASSWORD=12345 -d mysql
```
```shell
docker pull redis
```
### master

```shell
sudo docker run -it -v /home/Me/docker/chai/redis-master.conf:/usr/local/etc/redis/redis.conf --name redis-master redis /bin/bash
```

### slave

```shell
sudo docker run -it -v /home/Me/docker/chai/redis-slave.conf:/usr/local/etc/redis/redis.conf --link redis-master:master --name redis-slave redis /bin/bash
```


### 设置开机启动

```sh
docker update --restart=always redis-master
```



## 注意

1.  确保master的bind是0.0.0.0
2.  确保slave要link master:master

## 参考

[Docker：创建Redis集群](https://lw900925.github.io/docker/docker-redis-cluster.html)