# Attention
> Don't over packed.

### 备注
可能目前没有能力去编写一个服务端框架，现在应该主要以熟练使用和了解一个游戏服务器框架为主。参考框架： cellnet, leaf

## Server Impl Details

### 登陆流程
* 玩家http登陆
* 登陆成功session写入redis，设置超时时间与超时机制
* 玩家以session连接socket
* 连接成功返回用户信息
* 登陆成功

### session
测试连后台api生成session
后台生成测试账号和session并插入redis

### 日志
* 控制台输出日志
* 日志文件

### Heartbeat


### protobuf的使用
参考:
* [Golang Protobuf Github](https://github.com/golang/protobuf)
* [Protobuffer on Github](
https://github.com/protocolbuffers/protobuf)