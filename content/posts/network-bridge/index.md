---
title: "在 Linux 中使用网桥 bridge"
slug: "network-bridge"
date: "2021-01-09T06:32:00+0000"
lastmod: "2025-01-16T09:17:58+0000"
draft: false
tags:
  - "Bridge"
  - "Linux"
  - "Network"
visibility: "public"
---
# 0X00 什么是网桥

普通的桥就是连接本来不通的路的基础设施，网桥就是用来本来不通的网的设备。但是这里介绍的不是真正意义上的物理设备，而是在 Linux 上创建的虚拟设备。Linux 上不仅可以创建虚拟网卡，也可以创建虚拟网桥，所以说我们在 Linux 环境下学习网络知识确实是一个不错的选择（除非你有钱买一堆物理设备🤣）。

> 桥接器（英语：network bridge），又称网桥，一种网络设备，负责网络桥接（network bridging）。桥接器将网络的多个网段在数据链路层（OSI模型第2层）连接起来（即桥接）。 -- [Wikipedia](<https://zh.wikipedia.org/zh-cn/%E6%A9%8B%E6%8E%A5%E5%99%A8>)

# 0X01 网桥有什么用

网桥的本质就是将互不连通的网卡接到一起，原则上只有这么简单的功能，具体在这个功能之上能做什么事就看我们自己了。

如果用过 Docker 的话可以看一下自己本地的网络拓扑，实际上 Docker 的容器间通信和外网访问也是用到了 bridge 的。

最直观的一种操作就是：用一台双网卡机器串在两台主机之间，用来监听甚至修改数据包。乍一看这好像是搞坏事要用的技术手段呐，的确。如果有人拿一个双网卡的小机器（比如树莓派）串在了你的光猫和路由器之间，那么连接到路由器的所有外网请求都会经过这台小机器。如果你的流量不加密，比如用http协议或者ftp协议这种，那别人是可以看到中间的所有内容的，包括登录的密码之类的。所以这也是“不要随便连接来路不明的 Wi-Fi” 一大理由。

当然也不只是做坏事，现在的好多 WAF（Web Application Firewall）都是可以支持 “透明模式部署” 的，也就是说把这种安全软件所在的机器串在你的网关和 web 服务器之间，就可以在不破坏网络拓扑的情况下做到安全防护。

如果你想用这种方式来做软路由、绕开学校的网络共享限制，或是做流量分析都是可以实现的。

# 0X02 实验环境

如果要跟着做一下的话，需要简单配置一下环境：

![](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/linux_bridge.png)

首先我们需要三台虚拟机，有些人用Virtualbox、VMware Workstation、有些人又在用VMware Fusion甚至有直接用 KVM 的，每个的操作都不太一样，我们最终都是需要有一套这样的环境。

三台虚拟机，server_1 有一张网卡，配置静态 ipv4 地址为 `192.168.1.1/24`，server_2 有两张网卡，分别与 server_1 和 server_3 相连，server_3 有一张网卡，配置静态 ipv4 地址为 `192.168.1.2/24`。当然如果虚拟机软件不能让两台机器直连的话可以通过在机器间加路由器或者交换机的方式将其连接起来。

此时 server_1 和 server_3 的 ip 虽然在同一网段，但是应该是不通的。

> 如果这时候你的网络已经通了，那证明虚拟机和虚拟路由器配置有误。有一点是需要确认的：如果你用到了虚拟路由器的话，路由器要禁用 DHCP 或是给两台路由器配置不同的网段例如 `1.1.1.0/24` 和 `1.1.2.0/24`

# 0X03 如何配置网桥

好的我们开始配置网桥，其实过程很简单，总共分四步：创建网桥，将 server_2 的 ens33 网卡连接到网桥上，将 server_2 的 ens34 网卡连接到网桥上，将网桥启用。

首先在 server_1 上执行 `ping 192.168.1.2`，此时网络是不通的，不要停，一直叫他 ping 就好，一会儿通了可以瞬间看到效果。

在 server_2 上执行

```sh
    ip link add name br0 type bridge     # 创建一个名为 br0 的网桥

    ip link set dev ens33 master br0     # 将 ens33 接到 br0 上
    ip link set dev ens34 master br0     # 将 ens34 接到 br0 上

    ip link set dev br0 up               # 启用 br0 这时网络应该通了

    ip link set dev ens33 nomaster       # 将 ens33 与 br0 断开连接
    ip link set dev ens34 nomaster       # 将 ens34 与 br0 断开连接

    ip link set dev br0 down             # down 掉 br0

    ip link del br0                      # 删除网桥
```