---
uid: null
source: null
aliases: null
tags:
  - mutex
  - unique_lock
  - 源码分析
  - cpp
created: 2022-07-04 11:00:51
updated: 2023-03-07 17:10:42
title: mutex 、unique_lock、lock_guard、conditional_variable 源码浅析
dg-publish: true
---

# Mutex 、unique_lock、lock_guard、conditional_variable 源码浅析

## Mutex 类

mutex 类继承自 `__mutex_base`，主要包括以下部分

1. 构造函数负责初始化 `pthread_mutex_t mutex `（结构体）
2. 析构函数负责清除结构体成员
3. 禁用拷贝构造，拷贝复制

```cpp
 class __mutex_base
  {
  protected:
    typedef __gthread_mutex_t			__native_type;

    __native_type  _M_mutex;

    __mutex_base() noexcept
    {
      // XXX EAGAIN, ENOMEM, EPERM, EBUSY(may), EINVAL(may)
      __GTHREAD_MUTEX_INIT_FUNCTION(&_M_mutex);
    }

    ~__mutex_base() noexcept { __gthread_mutex_destroy(&_M_mutex); }
#endif

    __mutex_base(const __mutex_base&) = delete;
    __mutex_base& operator=(const __mutex_base&) = delete;
  };

```

此外，`mutex` 在 `__mutex_base` 的基础上封装了 `lock` 与 `unlock` 方法，即对互斥量进行上锁与解锁操作，过程比较简单。

### Mutex 为什么需要禁止拷贝构造和赋值运算符？

具体见：[[Pages/Mutex类为什么需要禁止拷贝构造和赋值运算符？\|Mutex类为什么需要禁止拷贝构造和赋值运算符？]]

以下代码对互斥量测试，假如我们取消了 mutex 类禁止拷贝构造相关的代码，即允许 mutex 对象进行拷贝构造或者拷贝赋值，会发生什么？

```cpp
#include "header.h"

#include <mutex>
#include <pthread.h>

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
    b = m;
    m.unlock();
    pthread_create(&t1, NULL, f2, NULL);
    pthread_create(&t2, NULL, f2, NULL);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    cout << cnt;
}
```

使用 gdb 来调试以上程序

```
(gdb) p m
$1 = {<std::__mutex_base> = {_M_mutex = {__data = {
        __lock = 0, __count = 0, __owner = 0, 
        __nusers = 0, __kind = 0, __spins = 0, 
        __elision = 0, __list = {__prev = 0x0, 
          __next = 0x0}}, 
      __size = '\000' <repeats 39 times>, 
      __align = 0}}, <No data fields>}

// 执行lock之后
(gdb) p m
$3 = {<std::__mutex_base> = {_M_mutex = {__data = {
        __lock = 1, __count = 0, __owner = 7522, 
        __nusers = 1, __kind = 0, __spins = 0, 
        __elision = 0, __list = {__prev = 0x0, 
          __next = 0x0}}, 
      __size = "\001\000\000\000\000\000\000\000b\035\000\000\001", '\000' <repeats 26 times>, 
      __align = 1}}, <No data fields>}

// 由于b复制的m，lock标志位为1，所以再次调用b.lock()时，进入阻塞
(gdb) thread 2
[Switching to thread 2 (Thread 0x7ffff7a4e700 (LWP 8569))]
#0  __lll_lock_wait (
    futex=futex@entry=0x5555555631c0 <b>, private=0)
    at lowlevellock.c:52
52      lowlevellock.c: No such file or directory.

```

## unique_lock

- 控制 mutex 的所有权  
- 在构造的时候可以选择对给定的锁上锁（默认），或者延迟上锁，或者尝试上锁  
- 构造完之后，可以执行 lock,unlock 操作  
- 析构的时候解锁  
- 支持拷贝构造和拷贝赋值，转移所的所有权

>![note]  
>只有在多线程环境内 unique_lock 构造的时候才会上锁

```cpp
// 只有在多线程环境下，该变量才会被激活
static inline int
__gthread_active_p (void)
{
  static void *const __gthread_active_ptr
    = __extension__ (void *) &GTHR_ACTIVE_PROXY;
  return __gthread_active_ptr != 0;
}
```

## lock_guard

- 同 c++17 的 scoped_lock 一样，实现了 RAII 机制（ [[Pages/同步原语\|同步原语]] ）  
- 只有构造函数和析构函数，构造时上锁，析构时解锁

## condition_variable

```cpp
// condition_variable example
#include <iostream>           // std::cout
#include <thread>             // std::thread
#include <mutex>              // std::mutex, std::unique_lock
#include <condition_variable> // std::condition_variable
 
std::mutex mtx;
std::condition_variable cv;
bool ready = false;
 
void print_id(int id) {
	std::unique_lock<std::mutex> lck(mtx);
	while (!ready) cv.wait(lck);
	// ...
	std::cout << "thread " << id << '\n';
}
 
void go() {
	std::unique_lock<std::mutex> lck(mtx);
	ready = true;
	// cv.notify_all(); // 这是重点
    cv.notify_one();
}
 
int main()
{
	std::thread threads[10];
	// spawn 10 threads:
	for (int i = 0; i < 10; ++i)
		threads[i] = std::thread(print_id, i);
 
	std::cout << "10 threads ready to race...\n";
	go();                       // go!
 
	for (auto& th : threads) th.join();
 
	return 0;
}
```

- `cond.notify_one() ` 只有一个线程获得锁，执行，其他线程仍阻塞  
- `cond.notify_all()` 所有线程都获得锁，打印完，结束

### 源码分析

- `cv` 通常配合 `unique_lock` 一起使用  
- `unique_lock` 先对 `mutex` 上锁

```cpp
void print_id(int id) {
	std::unique_lock<std::mutex> lck(mtx);
	while (!ready) {
		cv.wait(lck);
	}
	// ...
	std::cout << "thread " << id << '\n';
}
```

在 `cv.wait(lck)` 内部，`cv` 先对自己的 `mutex` 上锁，因为 `cv` 同时有多个线程在执行，同时如果有的线程先退出，那么 `cv` 的 `mutex` 将会析构，从而使得其他线程崩溃，所以 `mutex` 要转成要给 `shared_ptr`  
`__unlock` 对传来的锁进行解锁， 然后将独占锁转移到一个临时变量上，这样跳出 scope 之后，这一段能够反复执行

```cpp
    template<typename _Lock>
      void
      wait(_Lock& __lock)
      {
	shared_ptr<mutex> __mutex = _M_mutex;
	unique_lock<mutex> __my_lock(*__mutex);
	_Unlock<_Lock> __unlock(__lock);
	// *__mutex must be unlocked before re-locking __lock so move
	// ownership of *__mutex lock to an object with shorter lifetime.
	unique_lock<mutex> __my_lock2(std::move(__my_lock));
	_M_cond.wait(__my_lock2);
      }

```

- 使用谓词的 wait 能够避免唤醒丢失和惊群现象。
- 发送方在接收方进入等待前发送通知。
- 惊群是说即使没有发送通知，仍然可能唤醒
<style> .container {font-family: sans-serif; text-align: center;} .button-wrapper button {z-index: 1;height: 40px; width: 100px; margin: 10px;padding: 5px;} .excalidraw .App-menu_top .buttonList { display: flex;} .excalidraw-wrapper { height: 800px; margin: 50px; position: relative;} :root[dir="ltr"] .excalidraw .layer-ui__wrapper .zen-mode-transition.App-menu_bottom--transition-left {transform: none;} </style><script src="https://cdn.jsdelivr.net/npm/react@17/umd/react.production.min.js"></script><script src="https://cdn.jsdelivr.net/npm/react-dom@17/umd/react-dom.production.min.js"></script><script type="text/javascript" src="https://cdn.jsdelivr.net/npm/@excalidraw/excalidraw@0/dist/excalidraw.production.min.js"></script><div id="mutex、unique_lock、lock_guard、conditional_variable_源码浅析_2023-08-07_1625.34.excalidraw.md1"></div><script>(function(){const InitialData={"type":"excalidraw","version":2,"source":"https://github.com/zsviczian/obsidian-excalidraw-plugin/releases/tag/1.9.12","elements":[{"type":"image","version":217,"versionNonce":2000830492,"isDeleted":false,"id":"3J3_0XQbNfIbXpBL4TA1X","fillStyle":"hachure","strokeWidth":1,"strokeStyle":"solid","roughness":1,"opacity":100,"angle":0,"x":-186.35010167917613,"y":-161.36587444698822,"strokeColor":"transparent","backgroundColor":"transparent","width":439.7142857142857,"height":354,"seed":1542480932,"groupIds":[],"frameId":null,"roundness":null,"boundElements":[],"updated":1691397140219,"link":null,"locked":false,"status":"pending","fileId":"9dc16b4b34b5fc53395f84f20b3ae6d23bc7941a","scale":[1,1]},{"type":"arrow","version":70,"versionNonce":2078134044,"isDeleted":false,"id":"3zIA-K8Fxa4VUWIo9hiWm","fillStyle":"hachure","strokeWidth":1,"strokeStyle":"solid","roughness":1,"opacity":100,"angle":0,"x":15.213163677014734,"y":-13.006920663934011,"strokeColor":"#e03131","backgroundColor":"transparent","width":102.31577019942432,"height":16.00001284950656,"seed":887625380,"groupIds":[],"frameId":null,"roundness":{"type":2},"boundElements":[],"updated":1691396928938,"link":null,"locked":false,"startBinding":{"elementId":"takwbcqO","focus":0.712294059816533,"gap":8.94737484580591},"endBinding":null,"lastCommittedPoint":null,"startArrowhead":null,"endArrowhead":"arrow","points":[[0,0],[-102.31577019942432,-16.00001284950656]]},{"type":"text","version":316,"versionNonce":352272548,"isDeleted":false,"id":"takwbcqO","fillStyle":"hachure","strokeWidth":1,"strokeStyle":"solid","roughness":1,"opacity":100,"angle":0,"x":-9.83945505242599,"y":-4.0595458181281,"strokeColor":"#e03131","backgroundColor":"transparent","width":240.5,"height":134.4,"seed":1722669468,"groupIds":[],"frameId":null,"roundness":null,"boundElements":[{"id":"3zIA-K8Fxa4VUWIo9hiWm","type":"arrow"}],"updated":1691396928938,"link":null,"locked":false,"fontSize":16,"fontFamily":3,"text":"如果不解锁，\n那么主线程可能阻塞在lk上\n主线程收到通知也没用\n因此，提前解锁\n让主线程执行到wait\n此时才能正常收到notification\n","rawText":"如果不解锁，\n那么主线程可能阻塞在lk上\n主线程收到通知也没用\n因此，提前解锁\n让主线程执行到wait\n此时才能正常收到notification\n","textAlign":"left","verticalAlign":"top","containerId":null,"originalText":"如果不解锁，\n那么主线程可能阻塞在lk上\n主线程收到通知也没用\n因此，提前解锁\n让主线程执行到wait\n此时才能正常收到notification\n","lineHeight":1.2,"baseline":130},{"type":"image","version":128,"versionNonce":262610332,"isDeleted":true,"id":"wL4CreDg","fillStyle":"hachure","strokeWidth":1,"strokeStyle":"solid","roughness":1,"opacity":100,"angle":0,"x":-348.924520762826,"y":230.9283197997553,"strokeColor":"transparent","backgroundColor":"transparent","width":312.5,"height":500,"seed":48868,"groupIds":[],"frameId":null,"roundness":null,"boundElements":[],"updated":1691397129751,"link":null,"locked":false,"status":"pending","fileId":"5c83678e1b1a43967a059f21a6900d8c4620d71d","scale":[1,1]},{"id":"98kUEGQa","type":"text","x":-107.54654970273896,"y":456.8403354907325,"width":9.375,"height":19.2,"angle":0,"strokeColor":"#e03131","backgroundColor":"transparent","fillStyle":"hachure","strokeWidth":1,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":null,"seed":875308964,"version":2,"versionNonce":298226844,"isDeleted":true,"boundElements":null,"updated":1691397115791,"link":null,"locked":false,"text":"","rawText":"","fontSize":16,"fontFamily":3,"textAlign":"left","verticalAlign":"top","baseline":15,"containerId":null,"originalText":"","lineHeight":1.2}],"appState":{"theme":"light","viewBackgroundColor":"#ffffff","currentItemStrokeColor":"#e03131","currentItemBackgroundColor":"transparent","currentItemFillStyle":"hachure","currentItemStrokeWidth":1,"currentItemStrokeStyle":"solid","currentItemRoughness":1,"currentItemOpacity":100,"currentItemFontFamily":3,"currentItemFontSize":16,"currentItemTextAlign":"left","currentItemStartArrowhead":null,"currentItemEndArrowhead":"arrow","scrollX":230.9196545278508,"scrollY":144.63341468258633,"zoom":{"value":2.077795964429402},"currentItemRoundness":"round","gridSize":null,"currentStrokeOptions":null,"previousGridSize":null,"frameRendering":{"enabled":true,"clip":true,"name":true,"outline":true}},"files":{}};InitialData.scrollToContent=true;App=()=>{const e=React.useRef(null),t=React.useRef(null),[n,i]=React.useState({width:void 0,height:void 0});return React.useEffect(()=>{i({width:t.current.getBoundingClientRect().width,height:t.current.getBoundingClientRect().height});const e=()=>{i({width:t.current.getBoundingClientRect().width,height:t.current.getBoundingClientRect().height})};return window.addEventListener("resize",e),()=>window.removeEventListener("resize",e)},[t]),React.createElement(React.Fragment,null,React.createElement("div",{className:"excalidraw-wrapper",ref:t},React.createElement(ExcalidrawLib.Excalidraw,{ref:e,width:n.width,height:n.height,initialData:InitialData,viewModeEnabled:!0,zenModeEnabled:!0,gridModeEnabled:!1})))},excalidrawWrapper=document.getElementById("mutex、unique_lock、lock_guard、conditional_variable_源码浅析_2023-08-07_1625.34.excalidraw.md1");ReactDOM.render(React.createElement(App),excalidrawWrapper);})();</script>