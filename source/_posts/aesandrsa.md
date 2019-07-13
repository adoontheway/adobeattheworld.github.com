---
title: cryptjs vs crypto++ 加解密(上篇)
author: ado
date: 2019-07-09 11:01:23
description: Nodejs与C++通讯加密调研
tags: [aes,rsa,nodejs,c++,websocket]

---

# Preface

接着调研WebSocket通讯协议加密的事宜。

做法是：

* 协议加密使用AES
* AES的密钥由RSA加密之后进行传递
* 前后端各自生成自己的密钥对，然后交换各自的公钥
* 用自己的私钥加密数据，对方则用公钥解密

# Reference

* [前后端API交互数据加密——AES与RSA混合加密完整实例](https://www.cnblogs.com/huanzi-qch/p/10913636.html)
* [WebSocket数据加密——AES与RSA混合加密](https://www.cnblogs.com/huanzi-qch/p/11010153.html)
* [jsencrypt](https://github.com/travist/jsencrypt)
* [Rhino support? #21](https://github.com/travist/jsencrypt/issues/21)
* [openssl API : rsa_private_decrypt](<https://linux.die.net/man/3/rsa_private_decrypt>)

# Contents

## 前端

### RSA库

前端加密用的是 jsencrypt,使用的版本是2.3.0， 官网的tag里面最新的版本是3.0.0rc，愿意做小白鼠的可以去试一下：

```sh
npm i --save jsencrypt@2.3.0
```

需要注意的是，我们这个的**nodejs** *native*里用的，而不是在浏览器中使用，所以**jsencrypt**只要导入进来，就会报 <u>navigator is not defined</u> 的问题，这个时候，需要去 *jsencrypt.js* 中修改一下源代码，参考的是 **jsencrypt** 在 github上的 [issue #21](<https://github.com/travist/jsencrypt/issues/21>)，也就是上面的第四条参考。

首先，去第73行左右，也就是初始运行会报错的地方的前面加上以下代码：

```js
if( typeof window === 'undefined' )
  var window = {};
if( typeof navigator === 'undefined' )
  var navigator = {};
```

加上这个之后 run 这里是不会报错了，但是在另一个地方会报错，也就是第3850左右，即 `ASN1.prototype.getHexStringValue = function ()` ,这里只需要将 **ASN1** 改为 **window.ASN1**就可以了。

### AES库

照旧，CryptJS

### 测试用例

```js

var rsaUtil = {
    bits : 1024,
    thisKeyPair : {},
    genKeyPair : function (bits=rsaUtil.bits) {
        let genKeyPair = {};
        rsaUtil.thisKeyPair = new JSEncrypt({default_key_size: bits,log:true});
        genKeyPair.privateKey = rsaUtil.thisKeyPair.getPrivateKey();
        genKeyPair.publicKey = rsaUtil.thisKeyPair.getPublicKey();
        return genKeyPair;
    },
    encrypt : function (plaintext, publickey) {
        if( plaintext instanceof Object) {
            throw Error('invalid content, String only')
        }
        // publickey && rsaUtil.thisKeyPair.setPublicKey(publickey);
        return rsaUtil.thisKeyPair.encrypt(plaintext)
    },
    decrypt : function (ciphertext, privatekey) {
        // privatekey && rsaUtil.thisKeyPair.setPrivateKey(privatekey);
        let decstring = rsaUtil.thisKeyPair.decrypt(ciphertext);
        return decstring;
    }
}


function testRsa() {
    let text = '没纸了';
    let keypair = rsaUtil.genKeyPair();
    let ciphertext = rsaUtil.encrypt(text,keypair.publicKey);
    let plaintext = rsaUtil.decrypt(ciphertext,keypair.privateKey);
    console.log('Private Key : ', keypair.privateKey);
    console.log('Public Key : ', keypair.publicKey);
    console.log('CipherText : ', ciphertext);
    console.log('Decrypted Text : ', plaintext);
}

testRsa();
```

Output:

> Private Key :  
>
> -----BEGIN RSA PRIVATE KEY----- MIICWwIBAAKBgGsgGKaHUbFj5YnUpaF1o8dqReXJZWE4UpPZE3A8wzlU9W8A9xxs gee2CmJDmscvn9rAzABXDH6L4+bT+Xk4MbPHxW4XOxmWX4fAMyeIChZKqC1Z4CB1 M/h1EzXMLWqVOc3YQNFiJg/7Ftcc1uwlgsOZlXI068m3f+26eMKHew6lAgMBAAEC gYASd6wIOVfJ/vC5PCF5njAn1phKLtf9VJpXxpplRPRa09yj0ETJb6NNcMKFgiYS pJEoQwhoHpmWz0LwriZZ0Wgv3dyNVM9wloullMjAKhOIl6/piDUq03j+YWjI5i9h RzTeG584C7HmDLuA6q5/mA4o2seP8jLC+yPsoL+nGiLgQQJBAMja2QXImvWeSKlr LRO7HI9wnrY25c+BMAx3aNJhGF9pnIbleGJggbW2jKeB2h+wwcfaRzdEQn5/jX/W XJ4JTRkCQQCIiXB3Qj6vV4RqVu8yYMUuLT4PxfuBT/K57XTPjj97DjPvZSTRUn/v sLzhp62tOs5z9ZDqeA3F9ujkY2joW3NtAkEAs1ZUeEmJss1Fa5/6e6eR9LFFxMR3 pTNgR4FG4rE3JdoSnI+/UXeB2VHJfLcAn1MfcuZ+t+YGyRPyr7YitCjx4QJANnRq UIYxLgl/nFHT0Inb2pDcwbm6l1ZlflYAUo9vLgr6F66FQp7eu6AHlRAlKoPbt0/n XjxHcf8J35zuSE3A+QJAGgm81eV1c6uXhm94weDsmerv2LqViHporQjXM2/SdWNf biHNiiCg4jUqHHeMmqaGuLHdsBsCfQoN2Zxg2HtAKg== -----END RSA PRIVATE KEY----- 
>
> Public Key :  
>
> -----BEGIN PUBLIC KEY----- MIGeMA0GCSqGSIb3DQEBAQUAA4GMADCBiAKBgGsgGKaHUbFj5YnUpaF1o8dqReXJ ZWE4UpPZE3A8wzlU9W8A9xxsgee2CmJDmscvn9rAzABXDH6L4+bT+Xk4MbPHxW4X OxmWX4fAMyeIChZKqC1Z4CB1M/h1EzXMLWqVOc3YQNFiJg/7Ftcc1uwlgsOZlXI0 68m3f+26eMKHew6lAgMBAAE= -----END PUBLIC KEY----- 
>
> CipherText :  St8GeWQ8UlhItOC+6TwNOM2nb0j5SFQDyNZpbU0FE0HEfbWhWyTfYshjhLl9FdyDyb/uC/XTr779qjnaUycEFkswwxJAmHbfYB3YRFvkr+M8YmAJGthJYdgInXHZA5CgJEVgBbCs+EP0Rcv8vI1Plffa/vzHF41WS3E2dHL1dYs= 
>
> Decrypted Text :  没纸了

这里需要注意的是，照抄那位大哥的代码，会出现加密和解释失败的问题，在新建JSEncrypt实例的时候，将log选项设置为true:

```js
new JSEncrypt({default_key_size: bits,log:true});
```

会发现，原先的代码里面的公钥和私钥都重复设置了：

> A key was already set, overriding existing.  jsencrypt.js:4216
>
> A key was already set, overriding existing. jsencrypt.js:4216

所以这里我把重新设置公私钥的地方都注释掉了：

```js
// publickey && rsaUtil.thisKeyPair.setPublicKey(publickey);

// privatekey && rsaUtil.thisKeyPair.setPrivateKey(privatekey);
```

连接服务端的测试代码：

```c++

function testCPPRsa(params) {
    let keypair = rsaUtil.genKeyPair();
    let aeskey = aesUtil.genKey();
    let cryptedAeskey = rsaUtil.encrypt(aeskey);
    console.log("AESKey : ", aeskey);
    console.log("AESKey after encrypt:",cryptedAeskey);
    let socket = new net.Socket({allowHalfOpen:true});
    socket.setTimeout(3000);
    socket.on('timeout',()=>{
        // console.log('socket timeout');
        socket.end();
    });
    socket.on('connect',()=>{
        console.log("Connected event...");
    });
    socket.on('ready',() => {
        socket.setKeepAlive(true);
        // console.log('ready');
        let res = socket.write(keypair.privateKey);
        console.log('Flushed0:',res);
        res = socket.write(cryptedAeskey);
        console.log('Flushed1:',res);
        // socket.end();
        // console.log(keypair.privateKey);
        // socket.write(cryptedAeskey);
        // socket.end();
    });
    socket.on('end',()=>{
        console.log('end');
        // console.log(aeskey);
        // console.log(cryptedAeskey);
        // let res = socket.write(cryptedAeskey);
        // console.log('Flushed1:',res);
    });
    socket.on('drain',()=>{
        console.log('drain event');
    });
    socket.connect(8888, '192.168.2.160',()=>{
        console.log("Connected...")
    });   
    socket.on('error', (err)=>{
        console.error(err);
    });
    socket.on('close',(evt)=>{
        console.warn(evt);
    });
}
```



## 服务端

接下来就是测试与服务端RSA算法的兼容性了。

服务端RSA用的是openssl的，遇到了一些问题：

main和上次差不多，还是用的boost::asio，只是上次的代码同一个socket同步读只会读一次，这一次将单个socket放到一个线程里面去操作了，代码：

```c++
try{
        boost::asio::io_service service_;
        tcp::endpoint endpoint_(tcp::v4(), 8888);
        tcp::acceptor acceptor_(service_, endpoint_);
        cout << "addr:" << acceptor_.local_endpoint().address() << endl;
        cout << "port:" << acceptor_.local_endpoint().port() << endl;
        for (;;) {
            tcp::socket socket_(service_);
            acceptor_.accept(socket_);
            cout << "client:" << socket_.remote_endpoint().address() << endl;
            Client c(&socket_);
            }

    }catch(std::exception& e){
        cerr << "Exception: " << e.what() << endl;
    }
            
```

 单个处理类的头文件：

```c++
#ifndef CLIENT_H
#define CLIENT_H

#include <iostream>
#include <boost/asio.hpp>
#include <boost/thread.hpp>
#include <boost/date_time/posix_time/posix_time.hpp>


using boost::asio::ip::tcp;
using namespace std;

//#define b64map "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
//base64空位填充符
#define b64pad '='

class Client
{
public:
    Client(tcp::socket *sock);
    ~Client();

public:
    void Accept();
    void RsaDecrypt();
    // openssl自带的base64解密，似乎不成功
    bool base64Decode(const string& input, string& output);
    // jsencrypt 的反解翻译
    string b64tohex(const string& input);

private:
    tcp::socket* sock;
    string key ;//RSA解密用的私钥
    string content;//需要解密的内容
	// base64字符集
    const string b64map = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
    // jsencrypt 要用到的字符集
    const string BI_RM = {"0123456789abcdefghijklmnopqrstuvwxyz"};
};

#endif // CLIENT_H
```

client.cpp:

```c++
#include "Client.h"
//#include <thread>
#include <boost/thread.hpp>
#include <iostream>

#include <openssl/rsa.h>
#include <openssl/err.h>
#include <openssl/pem.h>
#include <openssl/err.h>
#include <openssl/ssl.h>

#include <boost/archive/iterators/base64_from_binary.hpp>
#include <boost/archive/iterators/binary_from_base64.hpp>
#include <boost/archive/iterators/transform_width.hpp>

using namespace std;
Client::Client(tcp::socket *sock_)
{
    sock = sock_;
    boost::thread th(boost::bind(&Client::Accept,this));
    th.join();
}

Client::~Client()
{

}

void Client::Accept()
{
    try
    {
        while(true)
        {
            vector<unsigned char> buf(1024);
            size_t s = sock->read_some(boost::asio::buffer(buf));
            cout << "Read " << s <<"bytes from client complete." << endl;
            string s1(buf.begin(),buf.begin()+s);
            if(key.size() == 0)
            {
                key = s1;
            }else
            {
//                content = s1;
//                base64Decode(s1,content);
                content = b64tohex(s1);
                cout << "Key : " << key << endl;
                cout << "Content : " << content << endl;
                RsaDecrypt();
            }
//            string ss(buf.begin(), buf.begin()+s);
//            cout << ss << endl;
        }
    }catch(std::exception &e)
    {
        cerr << "Exception: " << e.what() << endl;
    }
}

void Client::RsaDecrypt()
{
    //加载错误信息
    SSL_load_error_strings();
    //加载错误信息
    ERR_load_crypto_strings();
    vector<unsigned char> out;
    try
    {
        RSA *rsa = RSA_new();
        BIO *keybio;
        keybio = BIO_new_mem_buf((unsigned char *)key.c_str(), -1);
        rsa = PEM_read_bio_RSAPrivateKey(keybio, &rsa, NULL, NULL);
        int len = RSA_size(rsa);
        out.resize(len);
        int ret = RSA_private_decrypt(content.size(), (const unsigned char*)content.c_str(), &out[0], rsa, RSA_PKCS1_PADDING);
        if (ret <= 0)
        {
            out.resize(0);
            cout << "Failure." << endl;
            
            //获取最新的错误码
            unsigned long errorcode = ERR_peek_last_error();
            cout << "Error Code:" << errorcode << endl;
            char errmsg[1024] = {0};
            char *tmp = NULL;
            //获取和打印错误信息
            tmp = ERR_error_string(errorcode,errmsg);
            cout << errmsg << endl;
        }else
        {
            string s(out.begin(),out.end());
            cout << "Success : " << s << endl;
        }

        BIO_free_all(keybio);
        RSA_free(rsa);
    }catch(std::exception& e)
    {
        out.resize(0);
        cout << e.what() << endl;
    }
}


bool Client::base64Decode(const string& input, string& output)
{
    typedef boost::archive::iterators::transform_width<boost::archive::iterators::binary_from_base64<string::const_iterator>, 8, 6> Base64DecodeIterator;
    stringstream result;
    try
    {
        copy(Base64DecodeIterator(input.begin()) , Base64DecodeIterator(input.end()), ostream_iterator<char>(result));
    }catch(...)
    {
        return false;
    }
//Cleanup:
    std::string data = result.str();
    int datasize = data.size();

    output = result.str();
    return !output.empty();
}
//翻译过来的jsencrypt : base64tohex
string Client::b64tohex(const string& input)
{
    int k = 0;
    int slop,v;
    vector<unsigned char> vs;
    for(int i=0; i< input.size(); i++)
    {
        if(input[i] == b64pad) break;
        v = b64map.find_first_of(input[i],0);
        if(v < 0) continue;
        if( k == 0)
        {
            vs.push_back(BI_RM.at(v >> 2));
            slop = v & 3;
            k = 1;
        }else if(k == 1)
        {
            vs.push_back(BI_RM.at((slop << 2) |(v >> 4)));
            slop = v & 0xf;
            k = 2;
        }else if(k == 2)
        {
            vs.push_back(BI_RM.at(slop));
            vs.push_back(BI_RM.at(v >> 2));
            slop = v & 3;
            k = 3;
        }else
        {
            vs.push_back(BI_RM.at((slop << 2)|(v >> 4)));
            vs.push_back(BI_RM.at(v & 0xf));
            k = 0;
        }
    }
    string res(vs.begin(),vs.end());
    return res;
}

```

联调前端输出如下，前端是在hex2b64里加了日志输出的：

> before hex2b64: 1424343c15c7345163a7a4f8a19b0067fa309ae2e1ea6cc96195e1aeb021a2080c5231c2331a1b054114c07f3c8906e7085049302d53fa2d3535d9b0eeb4d052682bf53c1b6289c405fb901424a147a8c9d7b3e9736facd23eeb6bd7bfb8d81632ae64fa09b04722b0ed21efa6abb39d2a3cecea6fef618d2cf2be75ee25d534 
>
> 
>
> after hex2b64: FCQ0PBXHNFFjp6T4oZsAZ/owmuLh6mzJYZXhrrAhoggMUjHCMxobBUEUwH88iQbnCFBJMC1T+i01Ndmw7rTQUmgr9TwbYonEBfuQFCShR6jJ17Ppc2+s0j7ra9e/uNgWMq5k+gmwRyKw7SHvpquznSo87Opv72GNLPK+de4l1TQ= 
>
> 
>
> AESKey :  g9unnHKOXIj36Ee3

服务端输出：

> addr:0.0.0.0
> port:8888
> client:192.168.2.18
> Read 886bytes from client complete.
> Read 172bytes from client complete.
> Key : -----BEGIN RSA PRIVATE KEY-----
> MIICXAIBAAKBgGx6zMVC1v2r5sUSoD1udXKmJD6hTItGa+JElo5LB6xIszcWpEAq
> Zk/7Opg8WNvaYePdu+muwKrHf7K1yBqFeIshKFs5fEEwDvOyB6P3E3nGFF/ghMVK
> Cf75MJoHVB/WkIpavVC7ekph9A7w7yM04qe/UW/4jNu8YVSwGLkCJEsrAgMBAAEC
> gYAMs1/wJIA5WPv+zMx4BMu5bZxGvOnSUJ9m8XLTHBO85xIcfAkk/hHWgWp90mrw
> 6qOVYyuSE4S5OfxIdyIDgOb48+nCIOIdSkpkS7xHbB1VFEoRjcVtoKzQjxQO71tH
> cq0DBTh2ITjcKfrL/mJO7Lju+3r79WaRuv3v2TRvRi7uGQJBALchorA+6VmCpsD3
> q25J7hjUJsW5yWZb4uQ+FPsyRXdzkXQnpQYnpjbU3MYkkaHNhpbzgWN9eNi2wdch
> A653lBUCQQCXpOWfWge33TAk/ZUHz62WQiFJzZKwpvZIxiU/CcVal/6ZIZ60/VE7
> RW6EIRqZMV13LFa7JNW1ZuJIKNZNj/I/AkEAjFHRfoo18sgRTbp4OMGdFpk4PDjn
> AO9XItwO54rVf3ml8gKOh/DJgyoHICvVCs2YWwGqGrmBZ0xhqp05XO1zIQJAL56U
> jOnUqqRGzhbjicRpmlnB4k8v1VFY3Zl7cMQ8uSopxSSNlgBGVfqDrfJHS/v4gcfm
> pHX4tjJMtKQ+R7UAOQJBALLBwYhZ/vic6DcQTWJIZOy/mnJYh8ijqi0CD/5JhoPD
> Q7mRZIyGEX26KUSNteCGwBE5DNy8lhjXIk2PY0nIY64=
> -----END RSA PRIVATE KEY-----
> Content : 1424343c15c7345163a7a4f8a19b0067fa309ae2e1ea6cc96195e1aeb021a2080c5231c2331a1b054114c07f3c8906e7085049302d53fa2d3535d9b0eeb4d052682bf53c1b6289c405fb901424a147a8c9d7b3e9736facd23eeb6bd7bfb8d81632ae64fa09b04722b0ed21efa6abb39d2a3cecea6fef618d2cf2be75ee25d534
> Failure.
> Error Code:67522668
> error:0406506C:rsa routines:RSA_EAY_PRIVATE_DECRYPT:data greater than mod len
> Exception: read_some: End of file

所以现在纯粹是内容解密有问题了，密文内容过长。

这篇有点多，开个下一篇。