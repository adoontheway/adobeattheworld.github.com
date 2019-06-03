---
title: Websocket 二进制数据流 AES 加密
author: ado
date: 2019-05-25 10:29:19
description: websocket与 cocos creator通讯，对protobuf数据进行AES加密遇到的问题
tags: [js,websocket,c++]

---

# Preface

最近在对 protobuf 二进制数据进行aes加密操作，前端用的 [crypto-js](https://github.com/brix/crypto-js)，服务端用的 openssl 。解密之后的数据老是无法用 protobuf 对应的格式解析。

# Cause

经调查发现前端只能对字符串格式的数据进行aes加密，无法对二进制数据进行加密，所以前端对二进制数据做了一些特殊处理。而服务端 char是字符，也是 int8，所以影响不大。

所以前端基本是照抄了这个： 

[WebSocket 二进制传输 AES加密和解密](https://www.jianshu.com/p/659be55f8297)

# Content

## 想法一：直接将二进制数据转成string

由于前端二进制数据是 uint8array, 一个uint8对应的其实就是一个charcode，那么这个Uint8Array其实是可以直接字符串化的，所以可以先对二进制数据字符化，然后对这个字符串进行加解密，解密之后的数据再以charcode化就可以转成二进制数据了，试验如下：

```javascript
var CryptoJS = require("crypto-js");

var uint8arr = new Uint8Array(10);
for(var i = 0; i <= 9; i++)
{
    uint8arr[i] = Math.random()*0xff >> 0;
}
console.log("raw:",uint8arr);
console.log("======encrypt===============");
var str = "";
for(var i = 0 ;i <= 9; i++){
    str += String.fromCharCode(uint8arr[i]);
}
console.log("toString:",str);
console.log("length:",str.length)
var ciphertext = CryptoJS.AES.encrypt(str,"0000000000000000");
var enctext = ciphertext.toString();
console.log(enctext);
console.log("Length After Encrypto:",enctext.length);
console.log("======decrypt===============");
var rawStr = CryptoJS.AES.decrypt(enctext,"0000000000000000").toString(CryptoJS.enc.Utf8);
console.log("First Descrypt:",rawStr);
console.log("Equal to raw:",rawStr == str);
var rlen = rawStr.length;
var result = new Uint8Array(rlen);
for(var i = 0; i <= 9; i++)
{
    result[i] = rawStr.charCodeAt(i);
}
console.log("result:",result);
```

试验结果如下：

```sh
PS E:\workspace\node\crypto_test> npm start

> crypto_test@1.0.0 start E:\workspace\node\crypto_test
> node index.js

raw: Uint8Array [ 144, 246, 86, 140, 101, 132, 21, 240, 222, 249 ]
======encrypt===============
toString: öVeðÞù
length: 10
U2FsdGVkX19mC04Dy64U7xF5clPwdAA+qzpBCoFG6h/5msXIimk8lnur9vBDd5mB
Length After Encrypto: 64
======decrypt===============
First Descrypt: öVeðÞù
Equal to raw: true
result: Uint8Array [ 144, 246, 86, 140, 101, 132, 21, 240, 222, 249 ]
```

行是行得同，唯一要调查得是：

> CryptoJS.AES.decrypt(enctext,"0000000000000000").toString(CryptoJS.enc.Utf8);

其中 *toString* 传入的是 **CryptoJS.enc.Utf8**。

但是一个长度为 10 的字节数组加密后变成了一个长度为 64 的字符串，这个就有点可怕了。

## 想法二：老老实实的反解前端进行的一层封装



# Updates 

## 2019-5-29

今天在Go群里看到有大佬使用 CryptoJS 加密 openssl解不了的问题，后面据他调查的结果是：

> 是因为CryptoJS里面是用的evpkdf（这个鬼东西会生成一个salt 然后用salt来处理iv
>
> openssl 是最近的更新  hash 从 md5 变成了 SHA256  新版要指定-md md5（然鹅我是Liberssl啊

嗯，有时间还是要去研究一下。

我当初调试还是很肤浅的。