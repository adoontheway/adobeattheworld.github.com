---
title: setrlimit RLIMIT_NOFILE失败的问题
author: ado
date: 2019-05-23 16:57:33
description: 寻找 setrlimit(RLIMIT_NOFILE,&rlimit) 失败的问题
tags: [c++]
---

# Preface

```c++
struct rlimit rlmt;
if(getrlimit(RLIMIT_NOFILE, &rlmt) == -1)
        return false;
rlmt.rlim_cur = (rlim_t)655350;
rlmt.rlim_max  = (rlim_t)655359;
if (setrlimit(RLIMIT_NOFILE, &rlmt) == -1)
    return false;

return true;
```

上面的代码在 setrlimit 处返回了 -1 失败了。

系统环境：

> VMWare:
>
> - CentOS 7 64bit
> - Qt Creator 4.5.0 Based On Qt 5.10.0



## Content

## API

[来源：setrlimit() — Control maximum resource consumption](<https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.3.0/com.ibm.zos.v2r3.bpxbd00/rsrlim.htm>)

**RLIMIT_NOFILE**

> The maximum number of open file descriptors allowed for the process. This number is one greater than the maximum value that may be assigned to a newly created descriptor. (That is, it is one-based.) Any function that attempts to create a new file descriptor beyond the limit will fail with an EMFILE errno. An attempt to set the open file descriptors limit lower than that already used will result in an EINVAL errno.
>
> **Restrictions**
>
> This value may not exceed 524288.

这个值不应当超过 524288，修改为小于524288 的值之后，重新跑，失败。

## 修改系统设置

打开 /etc/security/limits.conf

插入：

```ini
*	soft	nofile 	2048
* 	hard	nofile	65555
```

保存，重新测试，失败。

## 解决

仔细阅读了一遍 api，发现这一行：

>  The hard limit can be raised only by a process which has superuser authority. 
>
> 硬限制只能由有超级用户权限的进程提升。

然后再命令行中用 sudo 来跑编译出来的结果，果然是可以了.....