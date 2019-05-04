---
title: Docker搭建Redis集群遇到的问题
date: 2019-05-04 14:25:12
tags: docker redis
---





[参考](https://lw900925.github.io/docker/docker-redis-cluster.html)

***redis容器中的redis.conf访问被拒***

```shell
sudo docker run -it -v /home/Landy/docker/chai/redis-master.conf:/usr/local/etc/redis/redis.conf --name redis-master redis /bin/bash
  
redis-server /usr/local/etc/redis/redis.conf
```
***Fatal error, can't open config file '/usr/local/etc/redis/redis.conf'***

```shell
sudo docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
ac746d069659        redis               "docker-entrypoint..."   8 minutes ago       Up 6 minutes        6379/tcp                            redis-master

```

```shell
docker logs redis-master
systemctl status -l docker.service
```

![Docker Redis cluster](E:\workspace\notes\adobeattheworld.github.com\source\_posts\assets\dockerredis-0.png)

***SELinux is preventing /usr/local/bin/redis-server from read access on the file redis-master.conf***
都是SELinux搞的鬼，编辑/etc/sysconfig/selinux 将selinux设置为disabled之后重启就可以了。

***error creating overlay mount to /var/lib/docker/overlay2/***

[参考](https://colobu.com/2018/06/28/Error-response-from-daemon-error-creating-overlay-mount-to-var-lib-docker-overlay2/)

### 重新建立mysql和docker容器

上面的解决方案中清理掉了所有的docker镜像和容器
```shell
docker pull mysql
sudo docker run --name chai_mysql -p 3306:3306 MYSQL_ROOT_PASSWORD=12345 -d mysql
```
```shell
docker pull redis
```
for master
```shell
sudo docker run -it -v /home/Landy/docker/chai/redis-master.conf:/usr/local/etc/redis/redis.conf --name redis-master redis /bin/bash
```

for slave
```shell
sudo docker run -it -v /home/Landy/docker/chai/redis-slave.conf:/usr/local/etc/redis/redis.conf --link redis-master:master --name redis-slave redis /bin/bash
```
注意：
1.  确保master的bind是0.0.0.0
2.  确保slave要link master:master