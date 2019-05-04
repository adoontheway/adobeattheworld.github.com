---
title: Boost问题手记
layout: post
tags: c++, boost
---

## multiple definition of 'xxxx'
在某个头文件中定义了一个函数，编译的时候报了这个错。
搜索了一下，加上static就好了。

## HRESULT:0X80041FE2
[https://forum.openframeworks.cc/t/exception-from-hresult-0x80041fe2-when-creating-new-project/27148/2](https://forum.openframeworks.cc/t/exception-from-hresult-0x80041fe2-when-creating-new-project/27148/2)
[https://social.msdn.microsoft.com/Forums/vstudio/en-US/18dea2e6-569c-471e-9345-dddbcdde1249/how-can-i-get-the-quotcommon-tools-for-visual-c-2015quot-in-visual-studio-express-2015-not?forum=vssetup](https://social.msdn.microsoft.com/Forums/vstudio/en-US/18dea2e6-569c-471e-9345-dddbcdde1249/how-can-i-get-the-quotcommon-tools-for-visual-c-2015quot-in-visual-studio-express-2015-not?forum=vssetup)
## Qt Creator: Multiple Definitions
.pro里面有同名文件，这个是新建文件覆盖的造成的，同时也是一个qt creator的小bug
[ttps://stackoverflow.com/questions/4964863/c-qt-multiple-definitions](tttps://stackoverflow.com/questions/4964863/c-qt-multiple-definitions)
##  ./bootstrap.sh遇到各种问题

*git submodule init* , *git submodule update*, update后执行./bootstrap.sh,执行成功的话会提醒执行 *./b2*
## pyconfig.h. No such file or directory

*find /usr -name pyconfig.h*没有
```
yum search python | grep python-devel
sudo yum install python-devel.x86_64
```
## asio找不到 undefined-reference-to-boostsystemgeneric-category-with-qt
boost大部分库只有头文件，有可能是安装了多个boost冲突了，需要去删除所有boost然后重新安装。
[https://stackoverflow.com/questions/19200911/undefined-reference-to-boostsystemgeneric-category-with-qt](https://stackoverflow.com/questions/19200911/undefined-reference-to-boostsystemgeneric-category-with-qt)
修改 *.pro*
```
INCLUDEPATH += /home/xushu/workspace/3rd/boost-1.66
LIBS += -L/home/xushu/workspace/3rd/boost-1.66/stage/lib/ \
    -lboost_system \
    -lboost_filesystem \
    -lboost_thread
```

## error while loading shared libraries: libboost_system.so.1.45.0: cannot open shared object file: No such file or directory
修改/etc/ld.so.conf 加入需要导入库的地址目录里，然后运行ldconfig即可，重启Qt生效
[https://stackoverflow.com/questions/4581305/error-while-loading-shared-libraries-libboost-system-so-1-45-0-cannot-open-sha
](https://stackoverflow.com/questions/4581305/error-while-loading-shared-libraries-libboost-system-so-1-45-0-cannot-open-sha
)

## /usr/lib64/librt.so.1:-1: error: error adding symbols: DSO missing from command line
.pro中 *LIBS += -lrt*