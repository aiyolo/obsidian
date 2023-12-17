---
uid: 
aliases: null
tags:
  - 面试/计算机网络
source: source:https://xiaolincoding.com/os/8_network_system/zero_copy.html
created: 2022-08-12 17:53:43
updated: 2023-03-02 19:08:40
title: 零拷贝与 DMA
dg-publish: true
---

# 零拷贝与 DMA

- DMA
   - 用途：专门用于 I/O 设备与内存之间传输数据，使得 cpu 不用参与数据搬运的工作，提高 cpu 的效率
   - 无 DMA：
	  - ![](https://raw.githubusercontent.com/aiyolo/imgrepo/main/img202303030031407.png)
   - 有 DMA：
	  - ![Pasted image 20220812183417](https://raw.githubusercontent.com/aiyolo/imgrepo/main/img202303030031536.png)
- 传统文件传输，如 read
   - 2 次 DMA 拷贝 +2 次 cpu 拷贝 +4 次用户态与内核态上下文切换
   - ![Pasted image 20220812183727](https://raw.githubusercontent.com/aiyolo/imgrepo/main/img202303030031656.png)
- mmap+write
   - 2 次 DMA 拷贝 +1 次 cpu 拷贝 +4 次上下文切换
   - ![Pasted image 20220812183948](https://raw.githubusercontent.com/aiyolo/imgrepo/main/img202303030031743.png)
- sendfile（不需要调用 read，write）^2592cf4a-3314-bf40
   - 2 次 DMA 拷贝 +1 次 cpu 拷贝 +2 次上下文切换
   - ![Pasted image 20220812184343](https://raw.githubusercontent.com/aiyolo/imgrepo/main/img202303030031842.png)
- 零拷贝，需要网卡支持 sg-dma
   - 查看是否支持：[[Pages/零拷贝与DMA#^d39c97\|零拷贝与DMA#^d39c97]]
   - 1 次 DMA+1 次 SG-DMA+2 次上下文切换
   - ![Pasted image 20220812193544](https://raw.githubusercontent.com/aiyolo/imgrepo/main/img202303030031938.png)
   - 优点：提高文件传输一倍以上性能
- 使用零拷贝的项目
   - kafka
- PageCache（内核缓冲区）^b7dcda9d-96d4-abeb
   - 磁盘高速缓存，缓存最近被访问的数据，读磁盘数据的时候先在 pagecache 中查找，如果没有找到，从磁盘中读取然后缓存再 pagecache 中
   - 缺点：不适合传输大文件，传输大文件时，内核将它们载入 pagecache，pagecache 很快占满，使得其他热点小文件无法充分使用 pagecache，其次大文件数据没有享受到缓存的好处，反而要多 dma 一次
   - 零拷贝技术基于 pagecache，针对大文件效率不高
- 大文件传输
   - 异步 IO+ 直接 IO（异步 IO 是指绕开 pagecache，直接从磁盘读到用户缓冲区，对于磁盘异步 IO 只支持直接 IO）
   - ![Pasted image 20220812194633](https://raw.githubusercontent.com/aiyolo/imgrepo/main/img202303030031036.png)

```cpp
❯ ethtool -k eth0 | grep scatter-gather  
scatter-gather: on  
		tx-scatter-gather: on  
		tx-scatter-gather-fraglist: off [fixed]

```
