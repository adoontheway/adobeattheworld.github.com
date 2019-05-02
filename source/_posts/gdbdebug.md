---
title: GDB 调试 Core文件
layout: post
---

# GDB Debug

### Open core file

```sh
> gdb <Program Excutable> <core file>
```

### Find Stack

```shell
(gdb) where
```

```shell
command class: stack

backtrace -- 打印所有堆栈帧
bt -- backtrace的简写
down -- Select and print stack frame called by this one
frame -- Select and print a stack frame
return -- Make selected stack frame return to its caller
select-frame -- Select a stack frame without printing anything
up -- Select and print stack frame that called this one
```

### Show threads

```shell
(gdb) info threads                 
```

### Use thread

```she
(gdb) thread 1
```

### Show all usable commands

```shell
(gdb) help all
```





