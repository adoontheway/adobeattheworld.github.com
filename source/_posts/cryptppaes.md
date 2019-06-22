---
title: Crypto++ AES加解密
author: ado
date: 2019-06-01 16:20:21
description: 使用Crypto进行AES加解密遇到的一些问题的笔记
tags: [c++, aes]
---

# Preface

之前遇到的 js-CryptoJS 与 c++ 进行AES加解密出现问题，今天再来试一下了。

# Content

## Cryptopp

嗯，放弃了openssl 的 aes 加解密接口，转而使用 Cryptopp 也就是 Ctypro++

### 安装

先从 [cryptopp On github](https://github.com/weidai11/cryptopp) 上下载源代码，然后再源代码目录下的 GNUmakefile 中第 75 行添加如下代码：

```makefile
CXXFLAGS += -pipe -fPIC
```

先执行：

```shell
make libcryptopp.a libcryptopp.so cryptest.exe
```

成功的话，再执行：

```sh
make install PREFIX=/usr/local
```

确保自己是 su 或者使用了 sudo。

确保起见，第二个命令执行成功之后再执行一下：

```sh
ldconfig
```

这样， Cryptopp 就成功的在系统里面安装了。

### 参考

[centos 64bit编译cryptopp](<https://www.lwlwq.com/post-cryptopp.html>)

[cryptoppwiki](<https://www.cryptopp.com/wiki/Linux>)

### 制作测试项目

在 Qt Creator 中新建一个项目，最好是纯 c++ 控制台项目。

在 .pro 中添加：

```ini
LIBS += /usr/local/lib/libcryptopp.a

LIBS += -lcurl -lcryptopp  -lsnappy  -lrt -lm -lz -ldl

INCLUDEPATH += /usr/local/include/cryptopp
```

测试代码 main.cpp 如下：

```c
#include <iostream>
// cryptopp
#include <iomanip>
#include <modes.h>
#include <aes.h>
#include <filters.h>

using namespace std;

int main()
{
    string keyStr = "0123456789abcdef";
    string ivStr = "abcdefghjklmnopq";
    CryptoPP::byte key[CryptoPP::AES::DEFAULT_KEYLENGTH],iv[CryptoPP::AES::BLOCKSIZE];
    memcpy(key, keyStr.c_str(),CryptoPP::AES::DEFAULT_KEYLENGTH);
    memcpy(iv, ivStr.c_str(), CryptoPP::AES::BLOCKSIZE);
//    memset(key,0x00,CryptoPP::AES::DEFAULT_KEYLENGTH);
//    memset(iv,0x00,CryptoPP::AES::BLOCKSIZE);
    string plaintext = "123456boy";
    string ciphertext;
    string decryptedtext;

    cout << "Plain Text (" << plaintext.size() << ") bytes"<< endl;
    cout << plaintext;
    cout << endl << endl;

    CryptoPP::AES::Encryption aesEncryption(key, CryptoPP::AES::DEFAULT_KEYLENGTH);
    CryptoPP::CBC_Mode_ExternalCipher::Encryption cbcEncryption(aesEncryption, iv);


    CryptoPP::StreamTransformationFilter stfEncryptor(cbcEncryption, new CryptoPP::StringSink(ciphertext));
    stfEncryptor.Put(reinterpret_cast<const unsigned char*>(plaintext.c_str()),plaintext.length());
    stfEncryptor.MessageEnd();

    cout << "Cipher text (" << ciphertext.size() << ") bytes" << endl;
    for(int i = 0; i < ciphertext.size(); i++)
    {
        cout << "0x" << std::hex << (0xff & static_cast<CryptoPP::byte>(ciphertext[i])) << " ";
    }
    cout << endl << endl;

    CryptoPP::AES::Decryption aesDecryption(key, CryptoPP::AES::DEFAULT_KEYLENGTH);
    CryptoPP::CBC_Mode_ExternalCipher::Decryption cbcDecryption(aesDecryption, iv);

    CryptoPP::StreamTransformationFilter stfDecryptor(cbcDecryption, new CryptoPP::StringSink(decryptedtext));
    stfDecryptor.Put(reinterpret_cast<const unsigned char*>(ciphertext.c_str()),ciphertext.size());
    stfDecryptor.MessageEnd();

    cout << "Decrypted Text :" << endl;
    cout << decryptedtext;
    cout << endl << endl;
    return 0;
}

```

Build -> Run, 结果如下：

> Plain Text (9) bytes
> 123456boy
>
> Cipher text (16) bytes
> 0xa5 0xf3 0x7e 0xb0 0x81 0x55 0xa7 0x8b 0x88 0x99 0x44 0xca 0x48 0xfd 0x87 0x14 
>
> Decrypted Text :
> 123456boy

### 参考

[example-of-aes-using-crypto:Stackoverflow](<https://stackoverflow.com/questions/12306956/example-of-aes-using-crypto>)

## CryptoJS

### 代码

```js
var CryptoJS = require("crypto-js");

function test1(){
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
    encrypted = encrypted.toString();
    console.log(encrypted);
}
test1();
```

调试的时候，可以跟进 `encrypted = encrypted.toString();` 这段代码，跟进到 enc-base64.js 文件里面：

```js
stringify: function (wordArray) {
	            // Shortcuts
	            var words = wordArray.words;
	            var sigBytes = wordArray.sigBytes;
	            var map = this._map;

	            // Clamp excess bits
	            wordArray.clamp();

	            // Convert
	            var base64Chars = [];
	            for (var i = 0; i < sigBytes; i += 3) {
	                var byte1 = (words[i >>> 2]       >>> (24 - (i % 4) * 8))       & 0xff;
	                var byte2 = (words[(i + 1) >>> 2] >>> (24 - ((i + 1) % 4) * 8)) & 0xff;
	                var byte3 = (words[(i + 2) >>> 2] >>> (24 - ((i + 2) % 4) * 8)) & 0xff;

	                var triplet = (byte1 << 16) | (byte2 << 8) | byte3;

	                for (var j = 0; (j < 4) && (i + j * 0.75 < sigBytes); j++) {
	                    base64Chars.push(map.charAt((triplet >>> (6 * (3 - j))) & 0x3f));
	                }
	            }

	            // Add padding
	            var paddingChar = map.charAt(64);
	            if (paddingChar) {
	                while (base64Chars.length % 4) {
	                    base64Chars.push(paddingChar);
	                }
	            }

	            return base64Chars.join('');
	        },
```



会发现，这里的 byte1,byte2,byte3  就是 Cryptopp 上面输出的：

```sh
0xa5 0xf3 0x7e 0xb0 0x81 0x55 0xa7 0x8b 0x88 0x99 0x44 0xca 0x48 0xfd 0x87 0x14 
```

## 结果

说明 CryptoJS 与 C++ Cryptopp 是可以相互之间加解密的。



## 问题

### \*** Error in `/home/workspace/bin/MyTest': double free or corruption (fasttop): 0x00000000025c3160 \***

运行上面的例子的时候出现这个报错，字面意思： 两次释放或者崩溃

## Update

由于之前不小心删掉了git上clone下来的cryptpp代码，且怀疑double free可能是使用了master版本所致，所以重新clone了分支，然后在目录下运行：

```sh
sudo make uninstall
```

来清理当前版本，成功之后，切换到最新的tag分支：

```sh
git checkout CRYPTOPP_8_2_0
```

之后按上次正常的流程安装。

跑上次的aes例子之后，果然没有报错了。

