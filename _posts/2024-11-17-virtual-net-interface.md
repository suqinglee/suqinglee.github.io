---
layout: post
title:  "Linux虚拟网络设备"
tags: [network]
---

* 目录
{:toc}

计算机系统通常由一个或多个网络设备组成，即eth0、eth1等。这些网络设备与物理网络适配器相关联，该适配器负责将数据包传输到线路上。

![](/assets/images/link/2.png)

而在虚拟网络中，需要在计算机系统内部，例如，在一台服务器上运行多个虚拟机或者容器，想要他们之间进行消息转发，需要借助虚拟网络设备。

## TUN/TAP

TUN/TAP允许用户空间程序，收发网络消息。它不是从物理介质接收消息，而是从用户空间程序接收消息；不是通过物理介质发送消息，而是写入用户空间程序。

它有TUN和TAP两种模式：

- TUN：工作在三层网络，用户程序读写的都是符合IP协议的消息。
- TAP：工作在二层网络，用户程序读写的都是符合以太网协议的消息。

![](/assets/images/link/3.png)

## Veth Pairs

veth从字面意思可以看出，是虚拟的二层网络接口，一般成对使用，可以看作的网线的两端，一端发送的消息可以在另一端接收。它可以在不同的网络命名空间中创建，实现两个网络命名空间之间的消息通信。

![](/assets/images/link/4.png)

## Bridge

Bridge（网桥），和物理网络中交换机的功能类似，连接二层网络设备，实现二层消息转发。

如下图，br0连接了物理二层设备eth0，和虚拟网络设备tap1、tap2，veth1，他们构成了一个二层网络。

![](/assets/images/link/1.png)

使用如下命令，构建上图网络拓扑：

1. 创建二层网络设备，类型为网桥，命名为br0。

```
ip link add br0 type bridge
```

2. 将主机物理二层网络设备，接入到网桥，这里master表示将eth0设置为br0的从属设备。

```
ip link set eth0 master br0
```

3. 将虚拟二层网络设备，也接入到网桥。

```
ip link set tap1 master br0
ip link set tap2 master br0
ip link set veth1 master br0
```

## Bond

Bond（绑定），可以将多个二层网络设备绑定成一个逻辑设备，以实现增加网络带宽、提供链路冗余备份或实现负载均衡等功能。

![](/assets/images/link/6.png)

使用如下命令，构建上图网络拓扑：

1. 创建二层网络设备，类型为bond，命名为bond0；`mode active-backup`表述主备模式；`miimon 100`表示每100毫秒，监测一次网络接口的链路状态。

```
ip link add bond0 type bond miimon 100 mode active-backup
```

2. 添加二层网络设备eth0、eth1到bond0中，不是只有物理设备可以，虚拟设备veth包括bond等也都可以添加到bond中。

```
ip link set eth0 master bond0
ip link set eth1 master bond0
```
