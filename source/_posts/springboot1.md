---
title: 来自Springboot的问候
author: ado
date: 2019-05-06 21:21:02
description: 今天头脑发热看了会儿Springboot教程的笔记记录
tags: [java, springboot]

---

# Preface

最近不知道为啥，头脑发热的看到了一篇 **Springboot** 教程。

# Steps

## 下载

去 [springboot initialzr](http://start.spring.io/) 选择： *Maven Project | Java* 

然后填写 **Group** : *com.xxxx* | **Artifact**: *demo*

然后点击 *Generate Project*。

之后会下载一个 **Springboot** 的初始脚手架的压缩包，

解压到你的工作空间即可。

## 导入

去[IntelliJ IDEA](https://www.jetbrains.com/idea/download/#section=windows)下载点下载社区版就可以了。

安装好之后，打开 **Idea** 导入项目：

*File -> New -> Module from Existing Source... ->* 选择上一步里面解压的目录，

接下来都选下一步就好了。

## 构建

其实之前走了一些弯路，后面导入就直接可以用了的。

双击打开 *pom.xml* ，点击 **Idea** 右边的 **Maven**，打开 **Maven** 视图。

在 **lifecycle** 中双击 **install**， **Maven** 会自动下载依赖库。

## 代码

照着教程去操作基本不会有任何问题发生。

只是在写测试用例的时候有个小小的问题，

以下代码有一些不一样：

```java
@Test
    public void getHello() throws Exception {
        mvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.content().string("Hello World."));
    }
```

源教程应该要直接导入：

```java
import org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
```

这里我们导入的是：

```java
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;
```

因为太久没有用过 java ，所以在跟着敲代码的时候就遇到了不理解 *status* 和 *content* 是什么方法的问题，源教程代码也只是贴了关键部分，没有贴导入部分的。

# Reference

[使用 Spring Boot 有什么好处](http://www.ityouknow.com/springboot/2016/01/06/spring-boot-quick-start.html)