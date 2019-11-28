---
title: 每天进步一点点-nginx
author: ado
date: 2019-11-28 14:23:27
description: 每天进步一点点，今天研究的是通过nginx来转发wss到ws
tags: [nginx,wss,ws]
---

# Contents

## 在CentOS上安装nginx
去官网[nginx.org](https://nginx.org)按照文档安装好**nginx**。  
注意，[nginx.com](https://www.nginx.com)是卖商业版nginx的，。  
安装成功之后，直接运行nginx就可以了，nginx的配置文件在 **/etc/nginx** 中。  
需要注意的是 **nginx** 需用以超级用户是身份启动，不然的话 **nginx** 相关日志会报错，没有权限访问，即使改了对应文件夹的权限也没用。    
我这里是启动失败的，输出日志是 **80** 端口被占用了，但是打开 *localhost:80* ，赫然显示的欢迎使用nginx，奇葩。  
发送个停止服务的命令试试：  
```sh
sudo nginx -s stop
```
嗯， **localhost:80** 访问不了了，说明是对的。

## 生成wss需要的key与证书
### 生成密钥
```sh
openssl genrsa -out privkey.pem 2048
```
注意，这里不能用：  
```sh
openssl genrsa -des3 -out privkey.pem 2048
```
如果用了的话，再nginx验证过程中会出现读取密码错误的问题：  
![读取密码错误](./nginx-ssl.png)  
### 生成证书请求文件
```sh
openssl req -new -key privkey.pem -out cert.csr
```
这里会生成一个证书请求文件，如果要正式上线的话，需要用这个去向第三方商业机构申请一个数字证书了。  
### 生成开发用的数字证书
```sh
openssl req -new -x509 -key privkey.pem -out cacert.pem -days 1095
```
以上可以生成一个自己开发用的证书了。

### 完成
现在就可以使用 **privkey.pem** 和 **cacert.pem** 了

## 内容提供
将所需要的内容从git上拉取到 **/usr/share/nginx/html/** 下面，这样需要在nginx展示的内容都放到这个子文件夹下面，假设叫 **sample** 吧。  

## conf修改
阅读了一遍 **etc/nginx/nginx.conf** ，知道了log的存放地址，看到了所有其他子配置都存放在 **/etc/nginx/config.d/** 下面。  
先修改 **default.conf** :
```conf
location / {
    root /usr/share/nginx/html/sample; #修改静态资源路径
    index index.html index.htm;
}
```

## 加入websocket.conf
然后我也有新增一个 **websocket.conf**:
```conf
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

upstream websocket {
    server localhost:8282; # appserver_ip:ws_port
}

server {
     server_name test.enzhico.net;
     listen 443 ssl;
     location / {
         proxy_pass http://192.168.1.110;
         proxy_read_timeout 300s;
         proxy_send_timeout 300s;

         proxy_set_header Host $host;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

         proxy_http_version 1.1;
         proxy_set_header Upgrade $http_upgrade;
         proxy_set_header Connection $connection_upgrade;
     }
    ssl_certificate /home/ado/works/cacert.pem; #前面生成的证书
    ssl_certificate_key /home/ado/works/privkey.pem; #前面生成的密钥
}
```

## 验证
重启 **nginx**：
```sh
sudo nginx -s reload
```
然后刷新页面就可以看到结果了。  

## 错误追踪
当然，我这个配置的过程中不是那么的顺利，nginx页面找不到的问题通过以下查看nginx的错误日志得到的：
```sh
sudo tail -100f /var/log/nginx/error.log
```
## 结果
目前暂时就进行到这里

# Reference
* [RHEL/CentOS上安装nginx](http://nginx.org/en/linux_packages.html#RHEL-CentOS)  
* [nginx反向代理WebSocket](https://www.xncoding.com/2018/03/12/fullstack/nginx-websocket.html)
* [nginx入门手册](https://nginx.org/en/docs)
* [openssl生成pem数字证书](https://blog.csdn.net/liangzhao_jay/article/details/8797423)
* [what-does-ssl-ctx-use-privatekey-file-problems-getting-password-error-indica](https://stackoverflow.com/questions/9380403/what-does-ssl-ctx-use-privatekey-file-problems-getting-password-error-indica)
