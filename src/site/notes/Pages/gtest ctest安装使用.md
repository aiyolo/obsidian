---
uid: null
aliases: null
tags: cpp,cmake
source: null
created: 2022-12-08 17:44:04
updated: 2023-03-08 15:44:29
title: Gtest Ctest 安装使用
dg-publish: true
---

# Gtest Ctest 安装使用

## Gtest 安装

源码编译安装

```shell
 7284  wget https://github.com/google/googletest/archive/release-1.8.1.tar.gz
 7285  tar -zxvf release-1.8.1.tar.gz
 7286  cd googletest-release-1.8.1
 7287  ls
 7288  cmake --build build
 7289  cmake -B build
 7290  cmake --build build
 7291  cd build
 7292  ls
 7293  sudo make
 7294  ls
 7295  sudo make install
```

或者命令行安装

```
sudo apt install libgtest-dev
```

## Gtest 使用

一个正常的 cpp 文件,引入 gtest 头文件

```cpp
#include <iostream>
// #include "src/TcpServer.h"
#include "../webssh/InetAddress.h"
#include "../webssh/Socket.h"
#include "../webssh/util.h"
#include "../webssh/TcpServer.h"
#include <gtest/gtest.h>

using namespace std;

TEST(InetAddressTest, testToPort)
{
    // Test a port number in the valid range (1-65535)
    {
        InetAddress addr(12345);
        std::string expected = "12345";
        std::string actual = addr.toPort();
        ASSERT_EQ(expected, actual);
    }
}


int main(int argc, char** argv) {
  ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

在 CMakeLists 文件，测试文件在 `add_executable` 之后还要和 `GTest::GTest` 库链接，当然前面不要忘了 `find_package(GTest REQUIRED)`

```
find_package(GTest REQUIRED)
foreach(file ${TEST})
    get_filename_component(file_name ${file} NAME_WE)
    message(STATUS ${file_name})
    add_executable(${file_name} ${file} ${SRC})
    target_link_libraries(${file_name} PRIVATE GTest::GTest)
    add_test(NAME ${file_name} COMMAND ${file_name})
endforeach(file ${TEST})
```

## 自定义输出

使用
```
std::cerr << "msg" << endl;
```

## CTest 和 GTest 的区别

gtest 是一个测试框架，而 ctest 可以批量执行可执行文件  
首先要 enable_ctesting()，然后将每个 test 可执行文件 add_test，语法如上、

## ASSERT_EQ 和 EXPECT_EQ 的区别

在 Google Test 中，ASSERT_EQ 和 EXPECT_EQ 是用来进行相等断言的两个宏，它们的主要区别是在断言失败时的处理方式不同。

- 当 ASSERT_EQ 断言失败时，程序会立即终止，并打印失败的断言信息。
- 当 EXPECT_EQ 断言失败时，程序不会立即终止，而是继续运行后面的测试代码，并在最后统一输出所有失败的断言信息。

因此，当我们需要终止程序并立即输出失败信息时，应该使用 ASSERT_EQ；当我们希望继续运行后面的测试代码并在最后统一输出失败信息时，应该使用 EXPECT_EQ。

## References

- [【CMake】使用ctest配置googletest_beidou111的博客-CSDN博客_cmake中ctest与gtest联合使用](https://blog.csdn.net/weixin_43940314/article/details/127673307?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EYuanLiJiHua%7EPosition-3-127673307-blog-90766192.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EYuanLiJiHua%7EPosition-3-127673307-blog-90766192.pc_relevant_default&utm_relevant_index=4)
- [matheusgomes28/cmake-google-tests: Skeleton CMake project that integrates Google Tests (github.com)](https://github.com/matheusgomes28/cmake-google-tests)
- [Integrating Google Test Into CMake Projects (matgomes.com)](https://matgomes.com/integrate-google-test-into-cmake/)
