---
aliases: null
tags: 
  - cpp
  - 源码分析
source: null
created: 2022-07-20 23:05:00
updated: 2023-03-02 12:44:19
uid: null
title: lower_bound()源码解析
dg-publish: true
---

# lower_bound() 源码解析

`lower_bound()` 可以通过二分法在数组里查找满足条件的某个数，它默认查找的是数组中第一个大于等于 `val` 的数。

如下面的这段代码将会输出 3；

```text
vector<int> arr={1,3,5,7};
cout << *lower_bound(arr.begin(), arr.end(), 3); // 打印3
```

还有一个 `upper_bound()` 函数与 `lower_bound()` 很相似，但是它默认返回的是数组中第一个大于 `val ` 的数。

自然而然的想到，能否利用这两个函数进一步找到数组中最后一个小于等于 `val` 的数，以及最后一个小于 `val` 的数。

为此我们对 `lower_bound()` 的源码稍作分析。

首先, `lower_bound()` 会调用更底层的一个函数 `__lower_bound()`，`lower_bound()` 默认的比较函数是 `__iter_less_val()`，即迭代器的值小于 `val` 时返回 true，我们也可以自己定义比较函数 `__comp(iter, val)`。

```text
    lower_bound(_ForwardIterator __first, _ForwardIterator __last,
		const _Tp& __val, _Compare __comp)
    {
      return std::__lower_bound(__first, __last, __val,
				__gnu_cxx::__ops::__iter_comp_val(__comp));
    }
```

在 `__lower_bound()` 里, 使用二分法进行查找,以默认的比较函数 `__iter_less_val` 为例进行说明

- 如果 `*__middle < __val` ,（即 `__comp(__middle,val)` 为 true），左边界向右边收缩
- 如果 `*__middle >=__val`, （即 `__comp(__middle,val)` 为 false）右边界向左边收缩

由于最后返回的是左边界的迭代器，左边的迭代器总是满足条件 2，而不满足条件 1

所以总体上看，`lower_bound()` 的效果在于，不断的向左收缩右边界，找到一个大于等于 `val` 的值

如果把默认的比较函数换成一般的函数 `__comp(iter，val)`，那么 `lower_bound() ` 相当于找到第一个 `!comp(iter, val)` 的值。

![](https://img-blog.csdnimg.cn/img_convert/f6db3f1f8b38a31688538778ed96afaf.jpeg)

而 `upper_bound()` 默认的比较函数是 `__val_less_iter()`，也可以自定义比较函数 `__comp(val,iter)`，它和 `lower_bound` 的区别主要在于这里的 val 是第一参数。

`upper_bound()` 返回第一个满足 `__comp(val, iter)` 的数，如使用默认的仿函数，将返回第一个满足 `val< *iter` 的数。

分析到这里，回到我们的问题，怎么找到数组中最后一个小于等于 `val` 的数，以及最后一个小于 `val` 的数？

注意到不管是 `lower_bound()` 还是 `upper_bound()`, 它们在满足了条件还是会不断从右向左收缩边界，最终返回左边界的值，对于 ` lower_bound()` 来说，这个条件是 `!comp(iter,val）`，而对于 `upper_bound() ` 来说，这个条件是 `comp(val, iter)`。

而要找到最后一个小于等于 `val` 的数，或者最后一个小于 `val` 的数，显然需要我们在满足小于等于条件时不断从左向右收缩边界，最终返回右边界的值。

为了做到这一点，可以使用 reverse_iterator！

剩下的就是把小于等于 `val` 的条件变换成 `!__comp(iter,val)`，看下面的变换方式

```
*iter<=val <====> !(iter>val) <=====> !greater(iter,val) <====> !greater<int>()
```

也就是说只要使用 ` greater<int>() ` 作为比较函数就可以实现查找数组里最后一个小于等于 ` val` 的数

同理 `upper_bound()` 搭配 reverse_iterator 和 `greater<int>() ` 能够实现在递增数组里查找最后一个小于 `val` 的值

最后是一个小测试

```cpp
int main(){
    vector<int> arr = {1,3,5,7,9,11,13,15};
    cout << *lower_bound(arr.rbegin(), arr.rend(), 9, greater<int>()).<< endl; // 返回9
    cout << *upper_bound(arr.rbegin(), arr.rend(), 9, greater<int>()) << endl; // 返回7
    for(auto arr_it = arr.rbegin(); arr_it != arr.rend(); arr_it++){
        cout << *arr_it << " ";
    }
}
```

related to [[Pages/lower_bound源码分析（草稿）\|lower_bound源码分析（草稿）]]