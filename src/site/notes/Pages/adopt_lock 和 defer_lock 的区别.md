---
uid: 
aliases: null
tags:
  - cpp/syntax
source: null
created: 2022-11-30 18:06:30
updated: 2023-03-02 12:30:23
title: adopt_lock 和 defer_lock 的区别
dg-publish: true
---

# adopt_lock 和 defer_lock 的区别

c++ 中 `lock_guard ` 通过 raii 机制实现了自动上锁和解锁互斥量，基本用法为

```cpp
{   
    static std::mutex io_mutex;
    std::lock_guard<std::mutex> lk(io_mutex);
    std::cout << "ddd" << std::endl;
} 
```

如果一段代码同时需要锁住两个互斥量, 不正确使用可能会引起死锁，比如再下面的代码中，先创建了两个 `unique_lock`，`defer_lock` 表示创建的时候不申请锁，后面人为的进行上锁

```cpp
❯ cat defer_lock.cpp
#include <bits/stdc++.h>
#include <thread>
#include <mutex>
using namespace std;
std::mutex mtx1, mtx2;
void f1(){
  {
	  // 因为后面要调用 lock 方法，所以使用 unique_lock 而不是 lock_guard
      std::unique_lock<std::mutex> lk1(mtx1, std::defer_lock); // 1 创建 lk1的时候不去上锁
      std::unique_lock<std::mutex> lk2(mtx2, std::defer_lock);// 2
      lk1.lock(); // 3 不能使用 lock(lk1), 它的参数至少要两个
      lk2.lock(); // 4 Risk of deadlock !
      std::cout << "ddd" << endl;
      // ...
  }
}

int main(){
  thread t1(f1);
  thread t2(f1);
  t1.join();
  t2.join();
}
```

如果两个 lock 分开执行，而且有两个线程，一个先执行了 `lk1.lock()` ，另一个线程先执行了 `lk2.lock()`，那么将会出现死锁

解决方法是对两个互斥量同时上锁

```cpp
❯ cat defer_lock.cpp
#include <bits/stdc++.h>
#include <thread>
#include <mutex>
using namespace std;
std::mutex mtx1, mtx2;
void f1(){
  {
      std::unique_lock<std::mutex> lk1(mtx1, std::defer_lock); // 1
      std::unique_lock<std::mutex> lk2(mtx2, std::defer_lock); // 2
      lock(lk1, lk2); // 3 同时上锁
      std::cout << "ddd" << endl;
      // ...
  }
}

int main(){
  thread t1(f1);
  thread t2(f1);
  t1.join();
  t2.join();
}
```

或者先同时上锁，再创建 guard，adopt_lock 要求调用线程当前拥有锁

```cpp
std::lock(mtx1, mtx2);
std::lock_guard<std::mutex> lock_a(mtx1, std::adopt_lock);
std::lock_guard<std::mutex> lock_b(mtx2, std::adopt_lock);
```

在 c++17 中引入了 `scoped_lock`， 它上面两种方法是等价的  

下面的三种方法是等价的

```cpp
{
	// use std::scoped_lock to acquire two locks without worrying about 
	// other calls to assign_lunch_partner deadlocking us
	// and it also provides a convenient RAII-style mechanism
	
	std::scoped_lock lock(e1.m, e2.m);
	
	// Equivalent code 1 (using std::lock and std::lock_guard)
	// std::lock(e1.m, e2.m);
	// std::lock_guard<std::mutex> lk1(e1.m, std::adopt_lock);
	// std::lock_guard<std::mutex> lk2(e2.m, std::adopt_lock);
	
	// Equivalent code 2 (if unique_locks are needed, e.g. for condition variables)
	// std::unique_lock<std::mutex> lk1(e1.m, std::defer_lock);
	// std::unique_lock<std::mutex> lk2(e2.m, std::defer_lock);
	// std::lock(lk1, lk2);
}
```

## 总结

下面三种方法在同时锁住多个互斥量时，是等价的

1. 先同时 lock，再创建 guard
2. 先创建 guard, 再同时 lock
3. 使用 scoped_lock

| Type            | Effect(s)                                                    |
| --------------- | ------------------------------------------------------------ |
| `defer_lock_t`  | do not acquire ownership of the mutex                        |
| `try_to_lock_t` | try to acquire ownership of the mutex without blocking       |
| `adopt_lock_t`  | assume the calling thread already has ownership of the mutex |

## 参考链接

1. [c++ - What's the difference between first locking and creating a lock_guard(adopt_lock) and creating a unique_lock(defer_lock) and locking? - Stack Overflow](https://stackoverflow.com/questions/27089434/whats-the-difference-between-first-locking-and-creating-a-lock-guardadopt-lock)
2. [std::scoped_lock - cppreference.com](https://en.cppreference.com/w/cpp/thread/scoped_lock)
3. c++ 并发编程实战
