---
aliases: null
tags:
  - leetcode
  - cpp
source: null
created: 2023-03-09 14:53:51
updated: 2023-03-20 19:40:19
uid: null
title: Leetcode-debug 中重要模块的设计
dg-publish: true
---

# Leetcode-debug 中重要模块的设计

#### 根据成员函数名调用成员函数

- 这个过程符合命令行设计模式
- `callMemFunc` 中，每个 `memfunc` 都是独立的，但是他们都有一个共同的 `class T`，而在注册的时候我们想要保存所有 `string` 到 `memfunc` 地址的映射，因此需要找到一个可以存放 `memfunc` 的类型
- 为此构造了一个 `memfunction` 的类，使它的成员变量 `m_func` 初始化为 `func` 的地址，而又由于 `m_func` 的类型随 `func` 的类型变换而变换，因此 `memfunction` 类必须要成为 `func` 类型 `F` 的一个模板类，从而可以根据不同的 `F` ，实例化成不同的 `memfunction` 对象
- 但是我们又不能做出一个 `map<string, MemFunction<F>>`, 因为 `F` 只有在 `callMemFunc` 中才能看到，因此需要让所有的 `MemFunction` 继承同一个 `MemFunctionBase<T>`, 而 `T` 是所有 `Memfunction` 都共有的，也就是实例的类型
- 现在 `MemFuction` 具有了两个模板参数，一个 `T` 代表类类型，一个 `F` 代表成员函数类型，注册成员函数的时候，把 `MemFuncition(m_func)` 存入到 map 里面，注意这里要存入指针，而且还要将其转换成它的父类指针，一方面是因为不创建指针则该实例是一个栈对象，出了函数作用域的话就销毁了，map 中也就没有了，另一方面，`map` 中无法知道 `F` 的具体模板参数，我们只能通过把它转换成它的父类函数才能存储起来
- 因为我们最终还是要调用 `memfunction` 子类中的对象的，因此必须要设计一个虚函数，使得父类指针访问虚函数的时候，能够调用子类中对应的覆盖的虚函数，为了方便直接使用 `operator() ` 做虚函数，该函数最终要调用的 `m_func`, 因此必须要给他传类的实例对象 `obj`, 以及成员函数参数 `m_argument`
- 这里 `m_arguments ` 依赖于 `F` 的类型，而 `memfunctionbase` 中要使用到该参数，因此必须设计成 `void*` 才行
- 剩下的就好办了

```cpp
template <typename T>
struct MemFunctionBase{
  virtual void operator()(T *obj, void* m_arguments) {} // 需要虚函数调用子类函数
};

template <typename T, typename F>
struct MemFunction:public MemFunctionBase<T>{
  using tuple_type = typename function_traits<F>::tuple_type;
  using return_type = typename function_traits<F>::return_type;
  static constexpr size_t I = function_traits<F>::arity;

  // tuple_type m_arguments;
  F m_func;

  MemFunction(F func):m_func(func){}

  // 输出位置
  template<size_t...Is>
  void exec(T* obj, void* m_arguments, index_sequence<Is...>){
    if constexpr (is_void_v<return_type>){
      (*obj.*m_func)(get<Is>(*(tuple_type*)(m_arguments))...);
    }
    else{
      cout << (*obj.*m_func)(get<Is>(*(tuple_type*)(m_arguments))...);
    }
  }
  void operator()(T* obj, void* m_arguments){
    exec(obj, m_arguments, make_index_sequence<I>());
  }
} ;
  template<typename F>
  void callMemFunc(string s, F&& func){
    using tuple_type = typename function_traits<F>::tuple_type;
    tuple_type tp;
    trans(m_arguments, tp);
    MemFunctionBase<T>* pFuncBase = static_cast<MemFunctionBase<T>*>(new MemFunction<T, F>(func)); 
    funcMap.emplace(s, pFuncBase);
    (*funcMap["shuffle"])(m_instance, (&tp)); // 转换成void指针
  }
```

## 出现过的 bug

该处出现内存泄漏

```cpp
template <typename T> struct MemberFunctionBase {
  virtual void operator()(T *obj, std::vector<std::string> &strArg) {
  } // 需要虚函数调用子类函数
  // virtual ~MemberFunctionBase(){}; //3.
};
std::unordered_map<std::string, MemberFunctionBase<T>*> funcMap; //2.

template <typename F> void registerMemberFunction(std::string s, F &&func) {
    // 把它们先上行转化，存入到funcmap里
    // 在调用的时候，通过调用父类的虚函数operator()()方法，来调用到子类的成员函数
    auto pFuncBase = static_cast<MemberFunctionBase<T> *>(new MemberFunction<T, F>(std::forward<F>(func))); // 1
    funcMap.emplace(s, pFuncBase);
  }
```

原因是 1 处申请的内存，没有释放，将子类指针上行转换父类指针后，删除父类指针，由于父类没有实现虚析构函数，子类空间不会删除

修改  
在父类添加虚析构函数，使用智能指针

```
virtual ~MemberFunctionBase(){}; //3.
std::unordered_map<std::string, std::shared_ptr<MemberFunctionBase<T>>> funcMap; //2
```

## 草稿

```cpp
template <typename T>
struct MemFunctionBase{
  virtual void operator()(T *obj) {} // 需要虚函数调用子类函数
};

template <typename T, typename R, typename...Args>
struct MemFunction:public MemFunctionBase<T>{
  R(T::*m_func)(Args...);
  tuple<decay_t<Args>...> m_arguments;
  MemFunction(R(T::*func)(Args...), tuple<decay_t<Args>...> tp):m_func(func), m_arguments(tp){}

  template<size_t...I>
  decltype(auto) exec(T* obj, tuple<decay_t<Args>...>& m_arguments, index_sequence<I...>){
    return (*obj.*m_func)(get<I>(m_arguments)...);
  }
  void operator()(T* obj){
    exec(obj, m_arguments, make_index_sequence<sizeof...(Args)>());
  }
} ;
template<typename R, typename...Args>
  void registerMemFunc(string s, R(T::*func)(Args...)){
    // auto func = to_function(forward(f));
    vector<int> a = {2,5,1,3,4,7};
    tuple tp = make_tuple(a, 3);
    MemFunctionBase<T>* pFuncBase = static_cast<MemFunctionBase<T>*>(new MemFunction<T, R, Args...>(func, tp));
    
    funcMap.emplace(s, pFuncBase);
    (*funcMap["shuffle"])(m_instance);
  }
  
// 或者只是使用一个typename 实现，也是可以的
template <typename T, typename F>
struct MemFunction:public MemFunctionBase<T>{
  using function_type = typename function_traits<F>::stl_function_type;
  using tuple_type = typename function_traits<F>::tuple_type;
  static constexpr size_t I = function_traits<F>::arity;
  tuple_type m_arguments;
  F m_func; // 注意这里只能使用F 不能使用function_type
  MemFunction(F func, tuple_type tp):m_func(func), m_arguments(tp){}

  template<size_t...Is>
  decltype(auto) exec(T* obj, tuple_type& m_arguments, index_sequence<Is...>){
    return (*obj.*m_func)(get<Is>(m_arguments)...);
  }
  void operator()(T* obj){
    cout << exec(obj, m_arguments, make_index_sequence<I>());
  }
} ;
