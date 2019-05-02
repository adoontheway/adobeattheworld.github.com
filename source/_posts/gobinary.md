---
title: Golang 字节流
layout: post
---


## Preface

这几天一直在研究怎么从 **golang** 实现的 *webscocket* 服务器上读取客户端传来的字节流。

由于自定义格式和避免沾包，所以需要实现自己的传输格式。之前用**java**，**lua**，**node**都是可以直接读取整个字节流的数据的。

我这边使用的通讯格式是 ：

|                   包长                   | 协议号 | 二进制序列包 |
| :--------------------------------------: | ------ | ------------ |
| 16   | 8    | n    |
| ---- | ---- | ---- |
|      |      |      |

**golang** 的字节操作是通过 *encoding/binary* 操作的，

```go
package test

import (
	"bytes"
	"encoding/binary"
	"fmt"
	"testing"
)

func TestBinary(t *testing.T)  {
	buf := new(bytes.Buffer)
	leng := buf.Len()
	a := 0
	var b int32 = 0
	c := 1
	d := false
	e := true
	binary.Write(buf, binary.LittleEndian,a)
	fmt.Printf("Len:%d, Cap:%d  [a := 0]\n",buf.Len()-leng,buf.Cap())
	leng = buf.Len()
	binary.Write(buf, binary.LittleEndian,b)
	fmt.Printf("Len:%d, Cap:%d  [var b int32 = 0]\n",buf.Len()-leng,buf.Cap())
	leng = buf.Len()
	binary.Write(buf, binary.LittleEndian,c)
	fmt.Printf("Len:%d, Cap:%d  [c := 1]\n",buf.Len()-leng,buf.Cap())
	leng = buf.Len()
	binary.Write(buf, binary.LittleEndian,d)
	fmt.Printf("Len:%d, Cap:%d  [d := false]\n",buf.Len()-leng,buf.Cap())
	leng = buf.Len()
	binary.Write(buf, binary.LittleEndian,e)
	fmt.Printf("Len:%d, Cap:%d  [e := true]\n",buf.Len()-leng,buf.Cap())
}
```

结果输出：

```go
=== RUN   TestBinary
Len:0, Cap:0  [a := 0]
Len:4, Cap:64  [var b int32 = 0]
Len:0, Cap:64  [c := 1]
Len:1, Cap:64  [d := false]
Len:1, Cap:64  [e := true]
--- PASS: TestBinary (0.00s)
```

可以看出，即使有类型推断，即使是非0的数字都是没法写入字节数组里面的；

*bool*值通过类型推断赋值的是可以写入字节数组。

写入之后的byte数组

![预览](../images/binarytest.png)

据*binary.Write* 的 **api** 描述，*Data* 必须是一个固定尺寸的值或者是固定尺寸的*slice*，或者是一个指针。

```
//Data must be a fixed-size value or a slice of fixed-size
// values, or a pointer to such data.
func Write(w io.Writer, order ByteOrder, data interface{}) error
```

从字节流中读取数值是一个相反的过程：

```go
var f int32
	var g,h bool
	binary.Read(buf, binary.LittleEndian,&f)
	binary.Read(buf, binary.LittleEndian,&g)
	binary.Read(buf, binary.LittleEndian,&h)
	fmt.Printf("%d,%v,%v",f,g,h)
```

> 以上输出: 0,false,true--- PASS: TestBinary (0.00s)

需要注意的是,data需要传入指针类型或者是固定尺寸的slice也可以。

```
//Data must be a pointer to a fixed-size value or a slice
// of fixed-size values.
func Read(r io.Reader, order ByteOrder, data interface{}) error
```

### 参考

[字节序及 Go encoding/binary 库](https://www.jianshu.com/p/1deed9012440)