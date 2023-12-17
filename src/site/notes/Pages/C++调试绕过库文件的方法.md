---
publish: true
aliases: null
tags:
  - cpp
source: null
uid: 202212311810
created: 2022-12-31 18:10:03
updated: 2023-03-02 12:27:18
dg-publish: true
title: C++ 调试绕过库文件的方法
---

# C++ 调试绕过库文件的方法

有时候我们想在调试 c++ 代码的时候, 只调试我们自己的代码，而不进入到库文件中。比如，在下面这段代码中，我们想在 `f()` 处执行 `step into` 会直接进入到 `display()` 函数体内，但是很不幸的是，直到 2022 年， c++ 对“Just my code”调试选项的支持都不是很好。

```cpp
#include "stdafx.h"
#include <functional>
#include <iostream>

void display() {
	std::cout << "Hello" << std::endl; // step into should go here
}

int main() {
	std::function<void()> f = &display;
	f(); // Step into here should go directly to "display()"
}
```

目前，我找到以下几个解决方案。

## 方案一：使用 Visual Studio 调试

使用 visual studio 在调试 c++ 代码时，默认启用了“just my code”选项，所以如果你是生成 Windows 平台的控制台程序，执行 step into 默认会跳过库文件。

但是这种方法，不支持本地计算机 Cmake 项目。

## 方案二：Visual Studio + Cmake

幸好，在 Cmake 中有选项支持对 Visual Studio debugger 启用“Just My Code” 支持。只需要在 CmakeLists. txt 文件中加入下面一行。

```
set (CMAKE_VS_JUST_MY_CODE_DEBUGGING 1)
```

但是，这种方法也仅支持本地计算机下的 Cmake 项目。如果你是要构建的远程 Linux 下的 Cmake 项目，仍然是不支持的。

其他的，还有从 gdb 的层面去解决 [^1] ，但是使用起来效果仍然不够理想.

## References

[^1]: [Skipping standard C++ library during debug session in gdb (reversed.top)](https://reversed.top/2016-05-26/skipping-standard-library-in-gdb/)

[^2]: [Ability to debug only my code (aka Just my code) : CPP-14618 (jetbrains.com)](https://youtrack.jetbrains.com/issue/CPP-14618)
