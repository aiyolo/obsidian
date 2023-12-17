---
uid: 
aliases: null
tags:
  - 工具效率
source: null
created: 2022-11-13 15:47:13
updated: 2023-03-08 15:56:06
title: Msys2 安装
dg-publish: true
---

# Msys2 安装

MSYS2 是一个工具和库的集合，为你提供一个易于使用的环境来构建、安装和运行本地 Windows 软件。

## 下载

[MSYS2](https://www.msys2.org/)

```
wget https://github.com/msys2/msys2-installer/releases/download/2022-10-28/msys2-x86_64-20221028.exe
```

## 安装 gcc

msys2 区分不同环境，如 ucrt, mingw64，不同环境对应一个目录

```
pacman -S mingw-w64-ucrt-x86_64-gcc
```

安装 mingw64 环境下的包

```
// 更新
pacman -Syu 
pacman -S --needed base-devel mingw-w64-x86_64-toolchain
pacman -S mingw-w64-x86_64-gcc
pacman -S mingw-w64-x86_64-toolchain
pacman -S mingw-w64-x86_64-cmake
// 删除
pacman -R package
```

```
pacman -S mingw-w64-ucrt-x86_64-toolchain
pacman -S mingw-w64-ucrt-x86_64-clang
pacman -S mingw-w64-ucrt-x86_64-clang-analyzer 
pacman -S mingw-w64-ucrt-x86_64-clang-tools-extra
pacman -S mingw-w64-ucrt-x86_64-lld
pacman -S mingw-w64-ucrt-x86_64-llvm
pacman -S mingw-w64-ucrt-x86_64-compiler-rt

```



