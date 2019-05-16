---
title: Golang的方法的测试
author: ado
date: 2019-05-16 18:57:16
description: Golang的对象方法和指针方法基准测试
tags: [go]
---

# Preface

今天下午开始在温习 [Go By Example](https://gobyexample.com/) ，温习到  [Method章节](https://gobyexample.com/methods) 的时候，发现了这么一句：

> Go automatically handles conversion between values and pointers for method calls. You may want to use a pointer receiver type to avoid copying on method calls or to allow the method to mutate the receiving struct.

大意是：

> Go在调用方法的时候会自从处理指针和对象之间的转换。最好是在方法调用的时候使用指针类型以避免复制 或者 允许方法对收到struct修改。

看起来，方法在调用的时候会有对象复制，两者在执行效率上应该有一定的区别。

于是，打算试验一下。

# Content

建立2个一模一样的结构体和一模一样的方法，唯一的区别只是结构体的名字不同而已，然后再 **bench** 中调用他们的方法，如下：

```go
package test

import "testing"

type A struct{}

func (a A) Hello() {}

type B struct{}

func (b *B) Hello() {}

func BenchmarkObjFunc(b *testing.B) {
	a := A{}
	for i := 0; i < b.N; i++ {
		a.Hello()
	}
}
func BenchmarkPtrFunc(b *testing.B) {
	b0 := &B{}
	for i := 0; i < b.N; i++ {
		b0.Hello()
	}
}

```

然后再 LiteIDE 中打开编译配置->自定义，找到 BENCHMARKARGS 再后面追加 -test.benchmem , 点击 Apply 然后直接再 LiteIDE 中运行 TestBench，结果如下：

```sh
BenchmarkObjFunc-12    	2000000000	         0.53 ns/op	       0 B/op	       0 allocs/op
BenchmarkPtrFunc-12    	2000000000	         0.26 ns/op	       0 B/op	       0 allocs/op
PASS
```

可知，指针方法确实比对象方法快一些，内存分配方面其实是没有区别的。