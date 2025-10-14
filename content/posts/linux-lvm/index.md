---
title: "LVM 的创建扩容与压缩"
slug: "linux-lvm"
date: "2022-09-22T13:56:00+0000"
lastmod: "2025-03-04T08:51:13+0000"
draft: false
tags:
  - "Linux"
  - "LVM"
visibility: "public"
---
# 0X00 介绍

> 阅读并了解 LVM 需要了解：Linux 基本操作、分区概念、文件系统概念

首先 LVM 的全称是 _Logical Volume Manager_ 逻辑卷管理。传统的方式是将一个磁盘分成类似于 `sda1/sda2/sda3` 的分区，然后再将这些分区格式化成类似于 `ext4/xfs` 这种文件系统，最后将文件系统挂载到某个目录上。但是这种方式下对磁盘空间进行重分配是比较麻烦的，将新安装的磁盘融入到现有系统中也是比较费力的，这就是 LVM 需要解决的问题。

总结下来 LVM 拥有这些功能

  1. LVM 可以方便的对现有逻辑卷进行压缩（初次分配多的空间不会浪费，可以压缩出来）
  2. 空闲的空间可以随时重新分配给逻辑卷（传统模式只能将空间分给最后一个分区，或者创建新分区）
  3. 新加入的磁盘也可以为其他逻辑卷扩容（传统模式并不方便为某个现有分区扩容）
  4. 可以将两块磁盘融合创建出一个更大的逻辑卷（两块 1T 磁盘可以创建出 2T 的分区）

特别需要注意的，RAID0 也可以将两个 1T 的磁盘合并为一个 2T 的，并且理论读写速度都会翻倍，但是这和 LVM 完全是两种不同的操作。

# 0X01 相关概念

开始使用 LVM 前需要先搞清楚它的几个基本概念

  * PV 是 Physical Volume 物理卷 — 从磁盘上分出来的物理分区
  * VG 是 Volume Group 卷组 — 多个 PV 组成的一个 Group
  * LV 是 Logical Volume 逻辑卷 — 从某个 VG 里创建出来的逻辑卷（可以格式化够挂载）
  * PE 是 Physical Extent 物理区域 — 是 PV 中最小的存储单元
  * LE 是 Logical Extent 逻辑区域 — 是 LV 中做小的存储单元

![LVM Cropped](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/09/15/lvm-cropped.jpg)

> 只是「简单用用」的话可以不管 PE/LE 这两个概念

# 0X02 测试环境

如果你想跟着练习一下的话，这里推荐使用虚拟机，新装一个随便什么 Linux 发行版（还是建议新一点，防止 LVM 版本过老）然后额外分配 2 块 10G 的虚拟磁盘就可以了（截图里是三块，因为我以为做完整个实验需要三块盘，结果最后没用上 😢）。

![SCR-20220922-m75](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/09/22/scr20220922m751.png)



![SCR-20220922-m87](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/09/22/scr20220922m87.png)

# 0X03 如何使用

我们已经知道 PV 就是一个物理分区、VG 是一堆 PV 的集合、LV 是从 VG 中划出来的逻辑卷了，现在开始用起来吧

其实这里说 PV 是一个物理分区是不严谨的，也可以不分区，直接对整体的块设备进行 `pvcreate` 的操作，后面为了方便我也会这么操作

## 从零创建

首先我们要通过 LVM 从零创建一个文件系统，就需要一路创建 PV、VG、LV 这三种

![SCR-20220922-n99](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/09/22/scr20220922n99.png)

这里可以看到，一共用到了 `pvcreate/vgcreate/lvcreate` 三个命令，最终创建出来的 lv 就已经是一个块文件了，接下来的格式化与挂载就不再赘述了，而且根据经验也可以猜到如果要删除的话就是 `lvremove/vgremove/pvremove` 了。

值得注意的一点是 `lvcreate -L 100M -n LV_NAME VG_NAME` 这行命令，表示创建一个 100M 的 LV，名字叫做 LV_NAME，从 VG_NAME 这个 VG 创建。

另一个值得注意的一点是，创建出来的 LV 是一个链接，是为了给 `dm-0/dm-1` 这种不好记的文件「重命名」，这样一来我们后面格式化、挂载之类的就可以用 `LV_NAME` 来操作了。

## 扩容

上面我们只创建了一个 100M 的分区，其实现在我们来给它扩容。首先我们可以看到 VG 明明有将近 10G 的空间，现在才用了 100M 显然是浪费的，先给这个 LV 扩容到 9G 再说

![SCR-20220922-nsd](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/09/22/scr20220922nsd.png)

上图中可以看到一共执行了这么几个步骤

  1. 检查当前 VG 的状态，看到还剩 9.9G 的空间处于 free 状态
  2. 将 LV 扩容至 9G （扩容命令跟创建差不多，只是没必要指定 VG 了，毕竟 LV 就已经和 VG 绑定了）
  3. 再度检查 VG 的状态，发现空间被划走了
  4. 使用 `fdisk -l` 看到那个块文件/分区的空间变大了

那如果这时候空间还不够用呢？这就体现出 LVM 的优势了，我们可以将一块新盘加入到 VG 中，继续给 LV 扩容

![SCR-20220922-o2o](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/09/22/scr20220922o2o.png)

上图中可以看到一共执行了这么几个步骤

  1. 创建一个新的 PV
  2. 将新的 PV 加入 VG
  3. 检查 VG 状态，发现 VG 扩容成功
  4. 将 LV 扩容至 18G
  5. 检查 LV 是否扩容成功

## 压缩

接下来就该给 LV 压缩了，例如同一个 VG 下的两个 LV 其中一个空间告急需要从另一个中挪一点空间出来，就可以这么操作（其实跟扩容没有区别）

![SCR-20220922-oh2](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/09/22/scr20220922oh2.png)

# 0X04 PE 和 LE

最后解释一下 PE(Physical Extents) 和 LE(Logical Extents) 两个术语。其中 PE 是 PV 的「最小存储单元」，LE 是 LV 的「最小存储单元」。

需要注意的是 PE 是创建 VG 的时候指定的，而并非创建 PV 的时候。而且 LE 的大小就是 LV 所在的 VG 的 PE 大小（这里有点绕，稍稍停下理解一下）。默认情况下 PE 是 4M，也就是说默认情况下创建出来的 LV 容量都将是 4M 的整数倍，当然也就可以解释下面这种「我明明创建的 99M 为啥变成了 100M」诡异情况了

![SCR-20220922-p16](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/09/22/scr20220922p16.png)

所以我们从原理上来说，创建 LV 的时候并不是「指定 LV 的容量为多少 M/多少 G」，而是「指定 LV 的容量为多少个 LE」

# 0X05 参考资料

[LVM - Debian Wiki](<https://wiki.debian.org/LVM>)
[LVM - ArchWiki（有中文）](<https://wiki.archlinux.org/title/LVM>)
