---
title: docker-compose快速搭建mongo与mongo-express
author: ado
date: 2019-09-07 17:07:48
description: docker-compose 快速搭建 mongodb 与 mongo-express控制台
tags: [mongo,docker]
---



# Preface

* [mongo official image](https://hub.docker.com/_/mongo)
* [mongo-express](https://hub.docker.com/_/mongo-express)

# Content

## yaml file

```yaml
version: '3.1'

services:
    mongo:
        image: mongo
        restart: always
        ports:
            - 27017:27017
        environment:
            MONGO_INITDB_ROOT_USERNAME: root
            MONGO_INITDB_ROOT_PASSWORD: 12345

    mongo-express:
        image: mongo-express
        restart: always
        ports:
            - 8081:8081
        environment:
            ME_CONFIG_MONGODB_ADMINUSERNAME: root
            ME_CONFIG_MONGODB_ADMINPASSWORD: 12345
            ME_CONFIG_MONGODB_ENABLE_ADMIN: 'true'
            ME_CONFIG_BASICAUTH_USERNAME: admin
            ME_CONFIG_BASICAUTH_PASSWORD: 12345
```

* ME_CONFIG_MONGODB_ENABLE_ADMIN : mongo-express是否需要登录验证
* ME_CONFIG_BASICAUTH_USERNAME,ME_CONFIG_BASICAUTH_PASSWORD: mongo-express 验证的账号密码

## command

```sh
docker-compose -f mongo.yaml up -d
```

使用:

```sh
docker ps
```

可以查看`mongo`和`mongo-express`是否正常启动起来了。

或者去掉 `-d` 查看运行日志。

## console

启动成功之后可以在` http://0.0.0.0:8081/` 中查看`mongodb`的状态和集合

## config

如果需要隐藏掉某些库的话，需要去`mongo-express`容器里面修改`config.js`：

```sh
docker exec -it mongo_mongo-express_1 bash
vi config.js
```

在 `config.js` 中找到 `blacklist`, 将需要隐藏的表加进去：

```js
blacklist: ['admin','config','local'],
```

`whitelist` 的设置刚好相反。