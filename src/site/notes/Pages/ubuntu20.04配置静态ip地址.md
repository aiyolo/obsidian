---
aliases: null
tags:
  - linux
source: null
created: 2020-07-10 21:49:00
updated: 2023-03-07 16:23:27
uid: null
title: Ubuntu20 .04 配置静态 Ip 地址
dg-publish: true
---

# Ubuntu20 .04 配置静态 Ip 地址

## 需求

因为我要连接 ssh，所以我需要一个固定的 ip。我的主机是用网线连上网络的，之前重启网络后，ip 一直都是固定的，但是今天 ip 却改变了，让我不得不搬一个显示器连上主机才知道改变后的 ip 地址，当然也可以局域网扫描 IP 获知新的地址。

下面介绍了两种方法，在 [https://assets.ubuntu.com/v1/37211e09-ubuntu-server-guide.pdf](https://link.zhihu.com/?target=https%3A//assets.ubuntu.com/v1/37211e09-ubuntu-server-guide.pdf) 可以找到，当然其他博客也能看到。

## 1、分配临时 Ip

先看一下原来 ip。

```text
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether a8:5e:45:52:26:31 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.190/24 brd 192.168.1.255 scope global dynamic noprefixroute eno1
       valid_lft 250811sec preferred_lft 250811sec
    inet6 240e:360:6915:9500:6017:87f7:11e6:182b/64 scope global temporary dynamic
       valid_lft 3451sec preferred_lft 3451sec
    inet6 240e:360:6915:9500:2edc:2015:1e5f:d23f/64 scope global dynamic mngtmpaddr noprefixroute
       valid_lft 3451sec preferred_lft 3451sec
    inet6 fe80::e75f:de2d:3d1f:3371/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

给以太网接口 eno1 添加一个 ip

```text
sudo ip addr add 192.168.1.22/24 dev eno1
```

启用这个 ip

```text
sudo ip link set dev eno1 up
```

再次查看 ip 地址,可以发现 ipv4 多了一个地址，可以 ping 通

## 2、分配固定静态 Ip

需要编辑/etc/netplan/**.yaml ,不同的电脑上名字好像不太一样

```text
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    eno1:
      addresses:
        - 192.168.1.22/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [114.114.114.114]
```

比如我的修改成了这样的，意思是在以太网接口 eno1 下，添加 ipv4 地址 192.168.1.22，网关 192.168.1.1，dsn 服务器 114.114.114.114。dsn 服务器是必须要填的，但是每一行的缩进不是严格要求的.

然后应用修改：

```text
sudo netplan apply
```

经过上面的配置，我的 ipv4 地址固定成了 192.168.1.22。
