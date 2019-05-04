---
title: C++学习笔记
layout: post
tags: c++
---

## Preface
> 这是最近学习C++, Boost, STL的相关笔记备份

## 关键字
* extern: 用extern关键字生命的全局变量和函数可以使得他们能够被跨文件访问
* _#pragma_: 预处理指令
* _#pragma pack_: 常用于处理不同协议的数据报文， eg, _#praga pack(1)_ 按照一字节的方式对齐
* \_1,\_2,\_3: 占位符，placeholder。用于_std::bind(fun,\_1,\_2) 或者boost::bind()
* 构造函数的冒号起分割作用。用于给成员变量赋值，初始化列表。适用于成员变量的常量const型。
* nullptr_t: 空指针字面量nullptr的类型
* NULL:实现定义的空指针常量
* register：修饰符，eg:register int miles，miles存储在寄存器而不是RAM的局部变量，大小受寄存器大小限制，且不能对他使用🏥运算符&,因为他没有内存地址。

## 方法
* strcpy(char \*dest,char \*src): 从src地址开始复制字符到dest地址，并返回指向dest的地址。
* getcwd(char \*buf,size_t size): 将当前工作目录的绝对路径复制到参数buf的内存空间
* atoi(str):字符串转数值
* strtoi(sctr,NULL,10):10表示10进制
* memset(void \*s, int ch,size_t n) 将s中当前位置后面的n个字节用ch替换并返回s。
* :: 双冒号用于访问静态成员。
* \. 和 \->: .号左边是实体， ->左边是指针
* &a : a的变量地址
* \*var
* <cmath> hypot(x,y): math.sqrt(x\*x+y\*y)
* <cmath> abs:返回整数的绝对值
* <cmath> fabs:返回十进制数的绝对值 

## 知识点

### 声明语法
```c++
修饰符 类型 变量名

class 类名:继承方式 基类名

type unsigned char BYTE;
type unsigned short WORD;
type unsigned long DWORD;
```

### 二进制运算
^ 异或 0^1=1, 1^1=0，即两者不同
~ 补码 ~0110=1001 ，即取反

### 函数
* 传值调用 void a(int x)
* 指针调用 void a(int \*x) : a(&b)
* 引用调用 void a(int &x)

### lambda
```c++
[capture](parameters)->return-type(body)
[](int x, int y)(return x < y)
[]{++global_x;}
[]（int x, int y)->int {int z = x+y; return z;}
```

* []:无传值
* \[x,&y\]：x传值，y传引用
* \[&\]:任何用到的外部变量隐式的以引用方式使用
* \[=]: 任何用到的外部变量隐式的以传值方式使用
* \[&,x]: x传值，其他引用
* \[=,&x]: x引用，其他传值
* \[=\]或\[&\]: 可以直接使用指针
* []： 必须显示的声明


### error
* 变量不明确：变量命名重复了，重新命名就好可以了。



###  allocating an object of abstract class type

出现这个的原因是有没有实现的虚函数，或者虚函数的实现方式有问题，参数或者返回值和母接口不符

### Q:HRESULT:0X80041FE2

A: 
https://forum.openframeworks.cc/t/exception-from-hresult-0x80041fe2-when-creating-new-project/27148/2
https://social.msdn.microsoft.com/Forums/vstudio/en-US/18dea2e6-569c-471e-9345-dddbcdde1249/how-can-i-get-the-quotcommon-tools-for-visual-c-2015quot-in-visual-studio-express-2015-not?forum=vssetup

###  Q：Qt Creator: Multiple Definitions

A: .pro里面有同名文件，这个是新建文件覆盖的造成的，同时也是一个qt creator的小bug
R:https://stackoverflow.com/questions/4964863/c-qt-multiple-definitions

### boost

Q: ./bootstrap.sh遇到各种问题
A：git submodule init , git submodule update, update后执行./bootstrap.sh,执行成功的话会提醒执行./b2

### pyconfig.h. No such file or directory

```sh
find /usr -name pyconfig.h没有
yum search python | grep python-devel
sudo yum install python-devel.x86_64
```

### Q:asio找不到 undefined-reference-to-boostsystemgeneric-category-with-qt

A:boost大部分库只有头文件，有可能是安装了多个boost冲突了，需要去删除所有boost然后重新安装。
R：https://stackoverflow.com/questions/19200911/undefined-reference-to-boostsystemgeneric-category-with-qt
修改

```ini
#.pro
INCLUDEPATH += /home/xushu/workspace/3rd/boost-1.66
LIBS += -L/home/xushu/workspace/3rd/boost-1.66/stage/lib/ \
    -lboost_system \
    -lboost_filesystem \
    -lboost_thread
```

### Q:error while loading shared libraries: libboost_system.so.1.45.0: cannot open shared object file: No such file or directory

A:修改/etc/ld.so.conf 加入需要导入库的地址目录里，然后运行ldconfig即可，重启Qt生效
R：https://stackoverflow.com/questions/4581305/error-while-loading-shared-libraries-libboost-system-so-1-45-0-cannot-open-sha

### Q：/usr/lib64/librt.so.1:-1: error: error adding symbols: DSO missing from command line

A：**.pro** 中 **LIB += -lrt**



### multiple definition of...

在使用对*vector<custom_struct>*使用自定义sort方法的时候，发生了*multiple definition of function* 的问题。

头文件里头这么定义的：

```c++
struct A
{
    string B;
}

bool CompareByProp(const A a0, const A a1)
{
    ...
    return true
}

```

在 *cpp* 中：

```c++
vector<A> As;
As.push_back(....);
sort(As.begin(), As.end(), CompareByProp)
```

虽然在头文件里头加了

```c
#ifndef A_DEF_H
#define A_DEF_H
...
#endif
```

但是还是没有用。

最后的解决办法是在 *function* 前面加了个 *static* 修饰符。

[问题解决： multiple definition of XXX](https://blog.csdn.net/liyuefeilong/article/details/44071053)