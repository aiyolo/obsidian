---
uid: 
aliases: null
tags:
  - cpp
source: null
created: 2022-08-19 16:03:20
updated: 2023-03-02 14:14:08
title: 为什么 Open 函数 Flag 和 Mode 参数混用不会出错
dg-publish: true
---

# 为什么 Open 函数 Flag 和 Mode 参数混用不会出错

在 c 语言里创建文件的时候经常会用到 creat 或者 open 函数，

open 函数常使用的一个接口是

```cpp
int open(const char *pathname,int flags,mode_t mode)
```

其中：

- 第一个参数表示文件路径
- 第二个参数表示文件打开方式，可选值有 O_RDONLY, OWRONLY, OWRWR 等
- 第三个参数表示文件创建权限，可选值有 S_IRUSR, S_IWUSR 等

creat 函数的接口是

```cpp
creat(path, mode)
```

它等价于：

```cpp
open(path, O_WRONLY|O_CREAT|O_TRUNC, mode)
```

有时候会看到 _creat( path, O_CREAT|O_RDWR)_ 这样的代码，非常明显的可以注意到 O_CREAT 不是 mode_t 类型 参数，但是这样代码编译后又不会报错，让人心生疑惑

在 fcntl.h 文件里定义了 O_CREAT 等宏

```cpp
#define O_CREAT	   0100
#define O_RDWR  02
#define S_IXUSR	0100 
```

经过宏替换后，这些以 0 开头的数实际上表示八进制的数

不难计算出

```
O_CREAT | O_RDWR = 0100 | 02 = 0100 | 0002 = 0102
```

0102 和默认掩码 0022 作用就得到了 0100

所以 ` creat( path, O_CREAT|O_RDWR) ` 实际上相当于创建了一个只有用户执行权限的文件，并不会报错
