---
title: Golang AES遇到的问题
author: ado
date: 2019-05-26 16:23:29
description: Golang测试AES遇到的问题
tags: [go,aes]

---

# Preface

最近遇到了nodejs的三方库 cryptojs AES 加密的数据在 linux 中解密后不正确的问题。所以顺便研究了一下 golang 的AES。

# Code

```go
package main

import (
	"crypto/aes"
	"crypto/cipher"
	"crypto/rand"
	"encoding/hex"
	"errors"
	"fmt"
	"io"
	"sync"
)

var commonKey = []byte("0123456789abcdef")
var syncMutext sync.Mutex

func SetAeskey(key string) {
	syncMutext.Lock()
	defer syncMutext.Unlock()
	commonKey = []byte(key)
}

func AesEncrypt(plaintext string) (string, error) {
	block, err := aes.NewCipher(commonKey)
	if err != nil {
		return "", err
	}
	in := []byte(plaintext)
	leng := len(plaintext)
	if leng%16 != 0 {
		leng = leng/16*16 + 16
		leng = leng - len(plaintext)
		for i := 0; i < leng; i++ {
			in = append(in, 0)
		}
		leng = len(in)
	}

	cipherText := make([]byte, aes.BlockSize+leng)
	iv := cipherText[:aes.BlockSize]
	if _, err := io.ReadFull(rand.Reader, iv); err != nil {
		return "", err
	}
	cipher.NewCBCEncrypter(block, iv).CryptBlocks(cipherText[aes.BlockSize:], in)
	return hex.EncodeToString(cipherText), nil
}

func AesDecrpt(d string) (string, error) {
	ciphertext, err := hex.DecodeString(d)
	if err != nil {
		return "", err
	}
	block, err := aes.NewCipher(commonKey)
	if err != nil {
		return "", err
	}
	if len(ciphertext) < aes.BlockSize {
		return "", errors.New("ciphertext too short")
	}
	iv := ciphertext[:aes.BlockSize]
	ciphertext = ciphertext[aes.BlockSize:]
	// fmt.Println(len(ciphertext), len(iv))
	cipher.NewCBCDecrypter(block, iv).CryptBlocks(ciphertext, ciphertext)
	return string(ciphertext), nil
}

func main() {
	se, err := AesEncrypt("12345")
	fmt.Println(se, err)
	sd, err := AesDecrpt(se)
	fmt.Println(sd, err)
}

```

# Why

为啥要做这个笔记呢？

因为这里有个比较坑的点：

使用 AES-CBC 进行加密的数据的长度必须是 16 的倍数，也就是这段补0的代码：

```go
in := []byte(plaintext)
	leng := len(plaintext)
	if leng%16 != 0 {
		leng = leng/16*16 + 16
		leng = leng - len(plaintext)
		for i := 0; i < leng; i++ {
			in = append(in, 0)
		}
		leng = len(in)
	}
```

不补充到 16 的倍数的化，会报错：

> input not full blocks

报错处：

```go
func (x *cbcEncrypter) CryptBlocks(dst, src []byte) {
	if len(src)%x.blockSize != 0 {
		panic("crypto/cipher: input not full blocks")
	}
    ...
```

x.blockSize 也就是使用的 aeskey 的长度，为16.

# Reference

> 块密码自身只能加密长度等于密码块长度的单块数据，若要加密变长数据，则数据必须先被划分为一些单独的密码块。通常而言，最后一块数据也需要使用合适填充方式将数据扩展到符合密码块大小的长度。

[golang 中AES加密详解](https://studygolang.com/articles/7302)

