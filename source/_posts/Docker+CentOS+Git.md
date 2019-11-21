---
title: Docker上安装Git
layout: post
tags: [linux, git, docker]
description: 乘兴而起，败兴而归
---
# 起因
公司版本管理用的是svn，写个文档word和exce起步，多人协作容易冲突，这些文档还相当的不友善，导致冲突的解决很麻烦。很多东西都是可以用markdown摆平的。之前有centos搭建git服务的经历，这次尝试这用docker来搭建一个本地的git服务，最后把这个镜像导出以后随时拉下来用。

# 目的
利用docker搭建git+jenkins的ci框架和文档系统。

# 环境
* Windows 10家庭版x64：逼不得已的，家庭版只能庄docker toolbox
* centos


# Docker安装
直接照着官网安装就可以了

## 拉取centos镜像
可以通过docket拉取官方镜像
```sh
docker pull centos
```
centos镜像可以通过
```sh
docker search centos
```
找到一堆，我使用的是第一个，也就是star最多的那个，也即是官方的。
启动centos容器：
```sh
docker run -it --name cornfield -p 10080:3000 -v /E/workspace/docker/centos:/usr/local/cornfield centos
```
执行成功之后，会进入centos的命令界面，无论如何，我先来一发更新
```sh
yum update
```

## 安装依赖项
```sh

yum install curl policycoreutils openssh-server openssh-clients
systemctl enable sshd
systemctl start sshd
```
__Failed to get D-Bus connection: Operation not permitted__
[
Failed to get D-Bus connection](https://blog.51cto.com/lizhenliang/1975466)

## 安装gitlab
之前搭建过gogs的服务，这次用gitlab试试看。
```sh

curl -sS http://packages.gitlab.cc/install/gitlab-ce/script.rpm.sh | bash
yum install gitlab-ce
```
### 参考
[
Centos7.2搭建gitlab服务器](https://www.jianshu.com/p/e7b83dd85f9c)

### 一个卧槽
突然发现gitlab官方有现成的[docker镜像](https://hub.docker.com/u/gitlab)，如果只是搭建gitlab服务的话，直接用这个就可以了。其他官方有在维护3个gitlab镜像的。[
Install GitLab with Docker](https://docs.gitlab.com/ce/install/docker.html)



# Install docker
1. go to docker official site, Products->Docker Desktop->Download for Windows->Get Docker, remember:login first,otherwise you will find the Get Docker is missing
2.  Motherfucker, docker for Windows only available for Windows10 Profession/Enterpriuse/Education, but mine is family,so turn out to Docker toolbox
3. Get Docker Toolbox for Windows, it will auto install VirtualBox, Docker Client on you PC.
4. Waste too much time on boot2docker by command line, and download it directly save it to c:/user/USERNAME/.docker/machine/cache
5. docker search centos -> docker pull centos -> docker run -idt centos
6. run Kitematic, login and you'll find your container.
7. chose you container, click exec.
8. docker run -ti -v /d//workspace/docker/centos:/home/workspace centos, attention the disk synax of Windows
9. yum -y install gcc gcc-c++
10. set mount file in VirtualBox->settings->share folder

遇到的问题

* [sudo:command not found] https://zhuanlan.zhihu.com/p/35279658

## 安装虚拟机增强功能遇到了一系列的问题，安装gcc的时候发现没网络，原来是网络设置问题

https://unix.stackexchange.com/questions/78295/centos-no-network-interface-after-installation-in-virtualbox