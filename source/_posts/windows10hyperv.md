---
title: Win10家庭版开启Hyper-v的脚本
layout: post
tags: hyperv, vmware
---


参考[Windows 10家庭版如何添加Hyper-V虚拟机？](https://www.ithome.com/html/win10/374942.htm)

```powershell
pushd "%~dp0"

dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hyper-v.txt

for /f %%i in ('findstr /i . hyper-v.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"

del hyper-v.txt

Dism /online /enable-feature /featurename:Microsoft-Hyper-V-All /LimitAccess /ALL
```

但是开发用的VMWare Workstation与Device/Credential Guard不兼容，导致打不开VMWare。

参考:

[VMware Workstation 与 Device/Credential Guard 不兼容？](https://www.zhihu.com/question/64511903)

以管理员身份打开命令行工具，输入：

```pow
bcdedit /set hypervisorlaunchtype off
```

重启，解决。