---
title: Crypto++ 与 CryptoJs AES加解密
author: ado
date: 2019-06-22 16:39:02
description: CryptoJS AES 加密，然后通过socket向 C++服务端发送加密数据，服务端C++ 使用Crypto++ 解密
tags: [c++,aes]

---

# Preface

上次的CryptoJS与Openssl双方的AES算法不兼容的问题接着搞。

# 服务端

## 服务端Socket Server

服务端通过 boost::asio::ip::tcp 提供的方法监听客户端信息：

```c++
#include <iostream>
#include <vector>
#include <deque>

#include <modes.h>
#include <aes.h>
#include <filters.h>
#include <boost/asio.hpp>    

using namespace std;
using boost::asio::ip::tcp;

void decryptoStrMsg(vector<unsigned char> msg, size_t s);
void decryptBinMsg(vector<unsigned char> msg, size_t s);
int main()
{
	try{
        boost::asio::io_service service_;
        tcp::endpoint endpoint_(tcp::v4(), 8888);
        tcp::acceptor acceptor_(service_, endpoint_);
        while (true) {
            tcp::socket socket_(service_);
            acceptor_.accept(socket_);
            vector<unsigned char> buf(128);
            size_t s = socket_.read_some(boost::asio::buffer(buf));
            decryptoStrMsg(buf, s);//字符串加解密
            //decryptBinMsg(buf, s);//字节流加解密
        }

    }catch(std::exception& e){
        cerr << "Exception: " << e.what() << endl;
    }
}
```



## Test 1: 字符串的加解密

```c++
void decryptoStrMsg(vector<unsigned char> msg, size_t s){
    string keyStr = "0123456789abcdef";
    string ivStr = "abcdefghjklmnopq";

    CryptoPP::byte key[CryptoPP::AES::DEFAULT_KEYLENGTH],iv[CryptoPP::AES::BLOCKSIZE];
    memcpy(key, keyStr.data(), CryptoPP::AES::DEFAULT_KEYLENGTH);
    memcpy(iv, ivStr.data(), CryptoPP::AES::BLOCKSIZE);
    CryptoPP::AES::Decryption aesDecryption(key, CryptoPP::AES::DEFAULT_KEYLENGTH);
    CryptoPP::CBC_Mode_ExternalCipher::Decryption cbcDecryption(aesDecryption, iv);
    string decryptedtext;
    CryptoPP::StreamTransformationFilter stfDecryptor(cbcDecryption, new CryptoPP::StringSink(decryptedtext), CryptoPP::StreamTransformationFilter::PKCS_PADDING);
    stfDecryptor.Put(&msg[0], s);
    stfDecryptor.MessageEnd();
    cout << "Decrypted Text: " << endl;
    cout << decryptedtext;
    cout << endl << endl;
}
```

## Test 2:字节流的加解密

```c++

void decryptBinMsg(vector<unsigned char> msg, size_t s){
    string keyStr = "0123456789abcdef";
    string ivStr = "abcdefghjklmnopq";
    CryptoPP::byte key[CryptoPP::AES::DEFAULT_KEYLENGTH],iv[CryptoPP::AES::BLOCKSIZE];
    memcpy(key, keyStr.data(), CryptoPP::AES::DEFAULT_KEYLENGTH);
    memcpy(iv, ivStr.data(), CryptoPP::AES::BLOCKSIZE);
    CryptoPP::AES::Decryption aesDecryption(key, CryptoPP::AES::DEFAULT_KEYLENGTH);
    CryptoPP::CBC_Mode_ExternalCipher::Decryption cbcDecryption(aesDecryption, iv);
    vector<unsigned char> decryptedtext;
    CryptoPP::StreamTransformationFilter stfDecryptor(cbcDecryption, new CryptoPP::VectorSink(decryptedtext), CryptoPP::StreamTransformationFilter::PKCS_PADDING);
    size_t t = stfDecryptor.Put(&msg[0], s);
    bool res = stfDecryptor.MessageEnd();
    for(int i = 0; i < decryptedtext.size(); i++){
        cout << (int)decryptedtext[i] << " ";
    }
    cout << endl;
}
```

# 客户端

## 发送加密的字符串

```js
var str = "123456boy";
    var key = "0123456789abcdef";
    var iv = "abcdefghjklmnopq";
    key = CryptoJS.enc.Latin1.parse(key);
    iv = CryptoJS.enc.Latin1.parse(iv);

    var options = {
        iv : iv,
        mode : CryptoJS.mode.CBC,
        padding : CryptoJS.pad.Pkcs7
    };

    var encrypted = CryptoJS.AES.encrypt(str, key, options);

    let data = CryptoJS.enc.u8array.stringify(encrypted.ciphertext);
    let socket = new net.Socket();
    socket.connect(8000, 'hostname',()=>{
        socket.write(data)
    })
```

## 发送加密的字节流数据

```javascript
var key = "0123456789abcdef";
    var iv = "abcdefghjklmnopq";
    key = CryptoJS.enc.Latin1.parse(key);
    iv = CryptoJS.enc.Latin1.parse(iv);

    var options = {
        iv : iv,
        mode : CryptoJS.mode.CBC,
        padding : CryptoJS.pad.Pkcs7
    };
    var len = 10;
    var uint8arr1 = new Uint8Array(len);
    for(var i = 0; i < len; i++)
    {
        uint8arr1[i] = i+1;
    }
    var contentWA = CryptoJS.enc.u8array.parse(uint8arr1);

    var encrypted = CryptoJS.AES.encrypt(contentWA, key, options);

    let data = CryptoJS.enc.u8array.stringify(encrypted.ciphertext);
    let socket = new net.Socket();
    socket.connect(8000, 'hostname',()=>{
        socket.write(data)
    })
```

## 工具方法

```js
const net = require('net');
var CryptoJS = require("crypto-js");

CryptoJS.enc.u8array = {
    stringify: function (wordArray) {
        var words = wordArray.words;
        var sigBytes = wordArray.sigBytes;
        var u8 = new Uint8Array(sigBytes);
        for (var i = 0; i < sigBytes; i++) {
            var byte = (words[i >>> 2] >>> (24 - (i % 4) * 8)) & 0xff;
            u8[i]=byte;
        }
        return u8;
    },
    parse: function(u8arr) {
        var len = u8arr.length;
        var words = [];
        for (var i = 0; i < len; i++) {
            words[i >>> 2] |= (u8arr[i] & 0xff) << (24 - (i % 4) * 8);
        }
        return CryptoJS.lib.WordArray.create(words);
    }
};

```

# 测试结果

## 结果

* 字符串加解密完全通过的
* 字节流解密之后会多出2个空字节，这个需要调查一下

所以整体结果是通过的。

# 其它

写了点小代码封装了一下 **Crypto++** 的代码，导出成库给其他应用使用。

在应用的 **pro** 文件中添加 *libcryptopp* 的库文件和 *include*文件地址，使用的时候无论如何都要报:

> undefined reference to vtable for xxx

最后解决的方法是，通过**Qt**提供的添加外部库的方式，将他以系统库的方式加入到项目中， **pro** 文件生成：

> unix:!macx: LIBS += -lcryptopp

因为我只需要在**linux**中使用，所以前面去掉了**windows**和**macos**。