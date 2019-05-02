---
title: Doxygen的安装
layout: post
---

**Doxygen**的的安装需要用到 *cmake*

```
yum install -y cmake
```

这个命令安装的是 *cmake 2.x* 版本，
而**doxygen**依赖的是 *cmake 3.x* ，
所以我们要安装的是
```
yum install -i cmake3
```
**doxygen**源代码的编译还依赖了其他一些库:*flex*, *bison*，这些可以直接通过*yum*安装的。
[官方安装教程](http://www.doxygen.nl/download.html)


### "font" is an unregitered media type
```shell

sudo update-desktop-database
```
```shell

grep MimeType /usr/share/applications/org.gnome.font-viewer.desktop
```
需要重启terminal

### 安装mysql-connector-c++报错，找不到openssl
因为没有装openssl-devel
```shell
yum install -y openssl-devel
```

**openssl和openssl-devel的区别**
* openssl是执行码部分
* openssl-devel包含了头文件，头文件参考，某些库文件等开发相关的东西

```shell
cmake ./mysql-connector-c++-8.0.12-src -DWITH_JDBC=ON -DCMAKE_INSTALL_PREFIX=/usr/local/mysql

```

```shell
cmake --build . --config release --target install
```
还是没有用，阅读官网文件发现jdbc需要编译的话，需要去手动通过*git submodule*去更新，于是放弃了当前不知谁提供的*mysql-connector-c++-8.0.12src**,去github上找到了对应的 [mysql-connector-cpp on github](*https://github.com/mysql/mysql-connector-cpp.git)
```
git clone https://github.com/mysql/mysql-connector-cpp.git
cd mysql-connector-app
git checkout 8.0.12
git submodule update --init

```

```shell
yum what provides libxxx*
```

又出了Could not find "libmysqlclient_r.a libmysqlclient.a"的问题

[mysql-connector-cpp官网安装文档](https://dev.mysql.com/doc/connector-cpp/8.0/en/connector-cpp-source-configuration-options.html)


### xxxxxqconfig.pri' has modification time 4711 s in the future
报这个错误的原因是因为vmware镜像是别人导出来的，然后本地的一些文件修改时间早于当前时间。
这次遇到的主要是QT/gcc_64/mkspec下一个文件以及QT/gcc_64/mkspecs/modules下面的所有文件，
针对单个文件运行
```shell
touch -m *.pri
```
文件夹下面的文件：
```shell
touch -m *.*
```
qt creator运行一下clean就可以正常build了。

###     qt: no such file or directory
*qt* 找不到某些头文件，但是在项目预览中明显是有的；
查看 *.pro* 文件，*HEADERS*里面也是有的；
但是 *INCLUDEPATH* 里面没有，在 *INCLUDEPATH* 里面加入相对路径之后就好了。

### qt creator: no rule to make target
复制别的项目，修改了项目的一些文件名，**build** 时出现了这个错误，**clean** 和重启 **qt** 都还存在这个问题。
删除**QT Creator** 编译生成的*build-[xxx]-Desktop_Qt_5_10_0_GCC_64bit-Debug*然后重新编译就可以了，
这个只是一个缓存问题。

### qt creator断点问题
qt creator断点进入的代码错误，这是因为进入的是so，而so里面的共享库代码是老版本的，导致用新库去调试项目的时候进入的方法对不上。我遇到的实际情况是，本来应该是调用一个返回stuct指针的方法，返回了一个值为4的int64，从而导致这个指针的地址直接是4而导致在访问这个struct的时候内存出错。

### cannot open shared object file:No  such file or directory
[参考](https://www.cnblogs.com/youxin/p/5116243.html)
Qt Creator构建出来的可执行文件找不到一些系统里面安装了的三方库，根据参考链接的提醒，去 _/etc/ld.so.conf_ 中加入so坐在的路径即可。
但是我看到 _/etc/ld.so.conf_ 的内容是这样的:
```c
include ld.so.conf.d/*.conf
```
也就是这个文件其实就比遍历了 _ld.so.conf.d_ 下的所有文件，
这些文件都是三方库的路径配置文件。
于是我在这个文件夹下面加了我的 _mongo.conf_ ,然后在其中写入我的 _mongocxx.so.\_noabi_ 的路径，然后执行：
```sh
sudo ldconfig
```
搞定。

### was not declared in this scope

经查证是形成了环形依赖
```c
//a.h
#include "b.h"

//b.h
#include "c.h"

//c.h
#include "a.h"
```

### jump to case label -fpermissive
作用域问题
```cpp
#define  a 1
#define b 2
```
```cpp
switch(hehe){
case a:
//bibobibobibo...
break
case b:
//bibobibobibo
break
}
```
上面由于case语句没有明确的大括号来标注他的作用范围，导致了很奇怪的问题，jump to case label只是其中之一，如果case语句里面有同名临时变量的话，也会产生redefined的问题，按照他的提示去掉redefined的定义后，还是会报其他错。解决方案就是把case语句的作用域用大括号包含起来，一直到break。
这个不像其他语言，case默认到break都是他的作用范围。

### std::vector reverse vs resize
reverse是预留
resize是重新分配大小

### vector.push_back是拷贝，不是引用传递

[
C++/STL_中的push_back方法与复制数据的问题](https://blog.csdn.net/u010003835/article/details/47442493)

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
### tree可以用来查看目录结构，但是需要安装
```shell
yum install -y tree

tree -d directory..
```
### docker配置redis集群

[参考](https://lw900925.github.io/docker/docker-redis-cluster.html)

### redis容器中的redis.conf访问被拒
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

![68665e56d3ada653a18d0fa2d52b25ad.png](en-resource://database/403:1)

***SELinux is preventing /usr/local/bin/redis-server from read access on the file redis-master.conf***
都是SELinux搞的鬼，编辑/etc/sysconfig/selinux 将selinux设置为disabled之后重启就可以了。

### error creating overlay mount to /var/lib/docker/overlay2/
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

### Valgrind
[使用 Valgrind 检测 C++ 内存泄漏](
http://senlinzhan.github.io/2017/12/31/valgrind/)