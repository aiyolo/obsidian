---
aliases: null
tags:
  - 物理
source: null
created: 2018-03-03 16:30:00
updated: 2023-03-07 16:16:12
uid: null
title: 为什么物理实验 B 类误差均匀分布时 sqrt(3)
dg-publish: true
---

# 为什么物理实验 B 类误差均匀分布时 sqrt(3)

参看 [中华人民共和国国家标准测量不确定度测量不确定度表示指南](https://link.zhihu.com/?target=https%3A//www.cnas.org.cn/fwzl/images/tc261sc1sysrkfjswyh/tzgg/2015/03/24/70E459F9EA361F6C3F4C675277B5CF3C.pdf) 有关 B 类不确定度的描述。

借用里面的例子：

![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/202303071605847.webp)

设物理量 $X$ 可能是区间 $[a,b]$ 范围的任意值，并且 $\Delta_{仪}=\frac{b-a}{2}$ 是半区间长度，假设物理量 $X$ 服从均匀分布 $X\sim U[a,b]$，那么 $X$ 的方差是  
$$Var[X]=\frac{(b-a)^2}{12}=\frac{\Delta_{仪}^2}{3}$$ 因此 B 类不确定度为
$$\mu_{B}=\sqrt{ Var[X] }=\frac{\Delta_{仪}}{\sqrt{ 3 }}$$
