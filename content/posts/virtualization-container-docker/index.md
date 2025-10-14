---
title: "虚拟化、容器、Docker"
slug: "virtualization-container-docker"
date: "2022-12-16T13:28:00+0000"
lastmod: "2025-03-04T08:56:06+0000"
draft: false
tags:
  - "Virtualization"
  - "Container"
  - "Docker"
visibility: "public"
---
# 0X00 虚拟化

首先**虚拟化 Virtualization** 它是一种技术，通过软件技术虚拟一张网卡（例如 Linux Bridge）、虚拟一个磁盘分区（例如 vmdk 文件）甚至直接虚拟一整台电脑（例如 VMware/VirtualBox）出来都是虚拟化技术的实装。

虚拟机就是通过虚拟化技术虚拟了一整套电脑所需的硬件，例如CPU、内存、磁盘、网卡等等，然后将它们拼在一起就是一台虚拟机了。

![vm](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/12/16/vm.jpg)

# 0X01 容器

区分这些的重点就在**容器 Container** 这里了。最早接触容器的时候很多人都说简单理解成轻量化虚拟机，不需要开关机、不需要操作系统、甚至是秒级启动的。当时我就死活不理解，明明是虚拟机为啥不需要操作系统不需要开关机的呢，其实但凡当时多解释几句，也就不会有这种困扰了。

> tips: 这里说的容器仅仅指代 Linux 容器

**容器的本质是进程** 。在 Linux 环境下创建一个进程，如果我们将它与其它进程隔离开，让它发现不了其它进程、再分配虚拟网卡、临时的文件系统等等等等，那它其实就是一个「容器」了。所以**容器的本质是进程** ，这样以来就说的通了。而且为什么不需要操作系统？因为它运行在一个现有的操作系统之上。为什么不需要开关机？因为它只是个进程。为什么是秒级启动？依旧因为它只是个进程～

关于 Linux 是如何将一个进程隔离开的，可以参考下面这一系列文章

  * [Docker基础技术：Linux Namespace（上）](<https://coolshell.cn/articles/17010.html>)
  * [Docker基础技术：Linux Namespace（下）](<https://coolshell.cn/articles/17029.html>)
  * [Docker基础技术：Linux CGroup](<https://coolshell.cn/articles/17049.html>)
  * [Docker基础技术：AUFS](<https://coolshell.cn/articles/17061.html>)
  * [Docker基础技术：DeviceMapper](<https://coolshell.cn/articles/17200.html>)

![virtualization-vs-containers](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/12/16/virtualizationvscontainers.png)

# 0X02 Docker

Docker 就简单了，它只是容器技术的一种实现，类似于 KVM 只是虚拟化技术的一种。不仅 Docker 是容器技术，还有 LXC、cri-o、containerd 等等一些。

# 0X03 总结

最后进行一个总结：**虚拟化技术是通过软件虚拟出硬件来的技术；虚拟机是虚拟化的集大成者，将一堆虚拟出来的硬件组装成一台电脑；容器技术的重点是将进程隔离出来，使之独立运行；Docker 则是容器技术的一种实现** 。
