---
aliases: null
tags:
  - 源码分析
  - cpp
  - stl
source: null
uid: 202210231606
created: 2022-10-23 16:06:45
updated: 2023-03-03 11:28:40
title: Deque 源码分析
dg-publish: true
---

# Deque 源码分析

#源码分析

`deque` 是标准库中的双端队列，从队列的前后位置都可以插入或者删除元素。`deque` 继承自 `_Deque_base` ，如下图所示，其数据成员包括：

- 一个主控区 `map`
- `start` 迭代器
- `finish` 迭代器

`map` 是一段连续区域，作为主控区，其中的元素节点指向不同缓冲区，每个缓冲区大小为 512 个字节。

`start` 和 `finish` 迭代器有四个数据成员，分别是

- `cur` 指向当前缓冲区中的元素
- `first` 指向当前缓冲区的头部
- `last` 指向当前缓冲区的尾部
- `node` 指向 `map` 中的节点，该节点指向缓冲区  
![Pasted image 20221023160700](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303031130064.png)  

`deque` 在头部尾部添加元素，如果缓冲区不足，那么要分配新的缓冲区

![Pasted image 20221023162229](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303031130065.png)  
![Pasted image 20221023162236](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303031130066.png)

其对应的重要源码部分见下面的代码注释。

```cpp

// deque 继承 _Deque_base

// _Deque_base_ 数据成员
protected:
      typedef typename iterator::_Map_pointer _Map_pointer;
      struct _Deque_impl
      : public _Tp_alloc_type
      {
		_Map_pointer _M_map;
		size_t _M_map_size; // 主控区
		iterator _M_start; // start 迭代器
		iterator _M_finish; // finish 迭代器


// 每个缓冲区 512 个字节 
#define _GLIBCXX_DEQUE_BUF_SIZE 512
  _GLIBCXX_CONSTEXPR inline size_t
  __deque_buf_size(size_t __size) // 返回每个节点的元素个数
  { return (__size < _GLIBCXX_DEQUE_BUF_SIZE
	    ? size_t(_GLIBCXX_DEQUE_BUF_SIZE / __size) : size_t(1)); }

// 迭代器 

struct _Deque_iterator
{

// 数据成员
  _Elt_pointer _M_cur;
  _Elt_pointer _M_first;
  _Elt_pointer _M_last;
  _Map_pointer _M_node;

  // 如果到了节点缓冲区尾部 
  // 那么node 指向下一个节点
  // cur 指向下一个节点缓冲区的起始位置
  _Self&
  operator++() _GLIBCXX_NOEXCEPT
  {
		++_M_cur;
		if (_M_cur == _M_last)
	  {
		_M_set_node(_M_node + 1);
		_M_cur = _M_first;
	  }
	return *this;
	  }
  }
};
```
