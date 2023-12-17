---
aliases: null
created: 2022-07-04 15:03:09
updated: 2023-03-07 16:56:17
uid: 202303071537
tags:
  - cpp/syntax
source: https://zhuanlan.zhihu.com/p/537368300
title: Mutex 类为什么需要禁止拷贝构造和赋值运算符？
dg-publish: true
---

# Mutex 类为什么需要禁止拷贝构造和赋值运算符？

---

假如我们取消了 mutex 类禁止拷贝构造相关的代码，即允许 mutex 对象进行拷贝构造或者拷贝赋值

```cpp
// stl_mutex.h

//   __mutex_base(const __mutex_base&) = delete;
//  __mutex_base& operator=(const __mutex_base&) = delete;

....
 // mutex(const mutex&) = delete;
  // mutex& operator=(const mutex&) = delete;
```

一个粗心的程序员可能写下如下的代码

```text
#include <stdc++.h>
#include <mutex>
#include <pthread.h>
using namespace td;

mutex m;
mutex b;
int cnt=0;

void *f2(void *){
    b.lock();
    for(int i=0; i<10; i++){
        cnt++;
    }
    b.unlock();
}
int main(){
    pthread_t t1, t2;
    m.lock();
    b = m;  // 给b赋值了一个上了锁的的互斥量
    m.unlock();
    pthread_create(&t1, NULL, f2, NULL);
    pthread_create(&t2, NULL, f2, NULL);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    cout << cnt;
}
```

以上代码会出现死锁，其原因在于，给 b 赋值了一个上了锁的的互斥量，相当于 b 已经 lock 了一次

gdb 可以查看 b 的成员,可以看到它的 __lock 成员为 1，并不是默认的 0

```text
$3 = {<std::__mutex_base> = {_M_mutex = {__data = {
        __lock = 1, __count = 0, __owner = 7522, 
        __nusers = 1, __kind = 0, __spins = 0, 
        __elision = 0, __list = {__prev = 0x0, 
          __next = 0x0}}, 
      __size = "\001\000\000\000\000\000\000\000b\035\000\000\001", '\000' <repeats 26 times>, 
      __align = 1}}, <No data fields>}
```

于是在子线程中再次对 b 上锁，会出现死锁。
