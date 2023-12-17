---
uid: null
aliases: null
source: null
created: 2022-04-09 22:34:37
updated: 2023-03-02 16:11:37
tags:
  - 操作系统
title: 最小的 Hello World
dg-publish: true
---

Related::: [[Pages/去除可执行文件的调试信息\|去除可执行文件的调试信息]]

# 最小的 Hello World

向屏幕打印 hello world，如果使用常见的写法会得到比较大的可执行文件，文件中含有不需要用到的符号表，启动文件，标准库等。

- `gcc -s hello.c -o hello.out` 可以去除符号表
- `_start` 函数是程序真正的入口，负责准备 main 函数需要的参数，调用 main 函数以及处理 main 函数的返回。使用 `gcc -e _start -nostartfiles` 可以指定程序入口，并且不使用启动文件
- 为了不使用标准库，可以直接使用 write 和 exit 系统调用

也就是向标准输出 write 一个字符串，其用 C 语言描述为：

```
write(fd, buf, count)  
exit(1);
```

其对应的汇编语言如下:

```bash
#include <sys/syscall.h>

.global _start ;程序入口为_start标号

_start:
movq $SYS_write, %rax ;使用系统调用SYS_write
movq $1, %rdi ; stdout的fd
movq $st, %rsi ; 字符串的位置
movq $(ed-st), %rdx ; 字符串的长度
syscall

movq $SYS_exit, %rax
movq $1, %rdi
syscall

st:
.ascii "hello world"

ed:
```

## References

- [编写一个最小的 64 位 Hello World - CJ Ting's Blog](https://cjting.me/2020/12/10/tiny-x64-helloworld/)
