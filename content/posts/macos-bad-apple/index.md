---
title: "苹果里的虫子：macOS 的几个臭毛病"
slug: "macos-bad-apple"
date: "2025-03-05T15:30:00+0000"
lastmod: "2025-03-18T07:16:31+0000"
draft: false
tags:
  - "macOS"
visibility: "public"
---
# 0X00 叠甲

开局先叠甲，我到现在为止用过一台 Mac mini，一台 Intel 的 MacBook Pro 和一台 M2 Max 的 MacBook Pro，自费购买 MacBook 花费超过 3W 元，累计使用超过 5 年，是个不折不扣的 macOS 用户。我认可很多 MacBook 和 macOS 的设计理念，如果我现在只能保留一台电脑，那极大可能我会选择一台 MacBook Pro。

😡 好了现在开始输出 😡

# 0X01 难用的包管理器

首先就是这个 `homebrew`，我在买 MacBook 之前看攻略的时候就看到很多人在吹 `homebrew` 有多好用，有多厉害，有多方便。他们的说辞一般是：一个命令就可以更新系统里很多的组件、一个命令就可以安装好开发环境、一个命令就可以XXXX。当时我作为一个 Archlinux 用户是有些懵逼的，但是看在这么多人吹捧的份上我也就相信了，心想着「这么多人都这样推荐，应该确实会很好用，如果能像 `pacman` 或者 `apt` 或者 `dnf` 这样的话那确实是很好的」。

结果呢？这个 `homebrew` 不光是个第三方工具，而且镜像源的数量也显著少于各个 Linux 发型版本。好不容易把它配置好之后，发现 `brew update/upgrade` 速度都明显比 `pacman/apt/dnf` 慢，而且仓库里的工具也显著少于常见的 Linux 发型版本。

我自己的体验（结合我自己的开发工作和环境）来说，Archlinux 使用的 `pacman` 属于独领风骚的， `apt` 和 `dnf` 则处在第二梯队，属于好用的，`brew` 则处于第三梯队，属于能用的。但确实不至于被拿出来吹。

> 如果所有吹 `homebrew` 的人都是在拿 macOS 和 Windows 对比的话，那我无话可说。`homebrew` 虽然不怎么好用，但是确实比 Windows 上没有包管理器来的更好。

# 0X02 不区分大小写的文件系统

下面这段是我在 Linux 中的实验结果：

```sh
    [shawn@archlinux ~]$ mkdir test_dir
    [shawn@archlinux ~]$ touch test_dir/hello_world
    [shawn@archlinux ~]$ touch test_dir/hello_WORLD
    [shawn@archlinux ~]$ touch test_dir/HELLO_world
    [shawn@archlinux ~]$ touch test_dir/HELLO_WORLD
    [shawn@archlinux ~]$ ls -l test_dir/
    total 0
    -rw-r--r-- 1 shawn shawn 0 Mar  6 14:27 HELLO_WORLD
    -rw-r--r-- 1 shawn shawn 0 Mar  6 14:27 HELLO_world
    -rw-r--r-- 1 shawn shawn 0 Mar  6 14:27 hello_WORLD
    -rw-r--r-- 1 shawn shawn 0 Mar  6 14:27 hello_world
    [shawn@archlinux ~]$
```

再下面这段是我在 macOS 中实验的结果：

```sh
    [shawn@mac] ~ $ mkdir test_dir
    [shawn@mac] ~ $ touch test_dir/hello_world
    [shawn@mac] ~ $ touch test_dir/hello_WORLD
    [shawn@mac] ~ $ touch test_dir/HELLO_world
    [shawn@mac] ~ $ touch test_dir/HELLO_WORLD
    [shawn@mac] ~ $ ls -l test_dir/
    .rw-r--r-- 0 shawn  6 Mar 14:28 hello_world
    [shawn@mac] ~ $
```

是不是很惊喜，是不是很意外？嘴上说自己是 Unix，但是真正 Unix 会使用的 ext4 和 XFS 甚至是 ZFS 等文件系统都是严格区分大小写的（ZFS 可配置，默认区分大小写），结果到了 macOS 上就不分了。也就是说如果你在 Linux 上将 `hello.py` 和 `HELLO.py` 两个文件打包，再到 macOS 中解压，就会原地消失一个 🫠

👇下面给大家表演一个「文件消失术」

```sh
    [shawn@mac] ~ $ ssh shawn@192.168.81.151
    Last login: Wed Mar  5 14:30:49 2025 from 192.168.81.1
    [shawn@archlinux ~]$ mkdir test_dir
    [shawn@archlinux ~]$ touch test_dir/hello_world
    [shawn@archlinux ~]$ touch test_dir/hello_WORLD
    [shawn@archlinux ~]$ touch test_dir/HELLO_world
    [shawn@archlinux ~]$ touch test_dir/HELLO_WORLD
    [shawn@archlinux ~]$ tar zcvf test_dir.tgz test_dir/
    test_dir/
    test_dir/hello_WORLD
    test_dir/HELLO_WORLD
    test_dir/HELLO_world
    test_dir/hello_world
    [shawn@archlinux ~]$
    logout
    Connection to 192.168.81.151 closed.
    [shawn@mac] ~ $ scp shawn@192.168.81.151://home/shawn/test_dir.tgz .
    test_dir.tgz                    100%  210   215.9KB/s   00:00
    [shawn@mac] ~ $ tar zxvf test_dir.tgz
    x test_dir/
    x test_dir/hello_WORLD
    x test_dir/HELLO_WORLD
    x test_dir/HELLO_world
    x test_dir/hello_world
    [shawn@mac] ~ $ ls -l test_dir/
    .rw-r--r-- 0 shawn  6 Mar 16:00 hello_world
    [shawn@mac] ~ $
```

# 0X03 随地大小便

说到 macOS 的随地大小便问题，哪怕不是 macOS 用户应该也有很多见过 `.DS_Store` 这个文件的吧。这个文件简单来说是 macOS 的文件管理器也就是 Finder 来创建的，文件中主要保存的就是当前目录的排序方式、展示方式等内容。如果你直接对这个目录进行打包（无论是在 GUI 还是 CLI 中），就会把这个文件直接包进去。如果是到 Windows 上解压，就会莫名其妙多出来一个 `.DS_Store` 文件；如果是在 Linux 上解压就更恶心了，它存在但默认又看不见 🙈

如果涉及到 git 仓库管理那就更愚蠢了，这个文件会随着日常使用经常产生变化，而且在多级目录下会出现大量的 `.DS_Store` 文件，只能通过 `.gitignore` 过滤掉，否则就会一直跟着你的代码库各种变动。

> 刚刚尝试在公司的代码库里 `find . -name .DS_Store | wc -l` 搜了一下，发现了整整 100 个 `.DS_Store` 🤷‍♀️

# 0X04 功能缺失，三方工具大行其道

macOS 其实有很多常用功能并不完善，所以诞生了很多小工具来专门给苹果擦屁股（其实 iOS 也一样）。下面举几个例子

## Finder

Finder 是 macOS 中的文件管理器，我之前用了那么多年的 Windows，也用了几年的 Linux（Gnome、KDE、Xfce 都长期用过），从来没想过安装一个第三方文件管理器，因为他们都足够好用。只有到了 macOS 上之后，没多久我就开始探索第三方的文件管理器，先后尝试过 QSpace 和 ForkLift。为什么呢？当然就是单纯的 Finder 不好用啊 👎

## Topbar

用过 Windows 的肯定都知道，打开的程序如果可以后台驻守的话会放在任务栏右侧，还可以通过系统设置让每个程序从固定显示、固定隐藏、条件显示三个状态里选一个。macOS 呢？完全没有，只要驻守在后台的程序全都在 Topbar 的右侧堆积着，你有三两个程序还好，十个八个的话就直接占据了一半的 Topbar，丑的很。重点是很多应用其实使用频率并不高，或者说都是通过快捷键调用的，从来不会在 Topbar 上去点，结果却要永久性占据一个位置。

而且说到这个就来气，新的 MacBook 给搞了个刘海，笔记本电脑，搞了个刘海，我不理解。最气的是这个狗屁刘海只有硬件知道，软件是不知道的。这意味着什么？意味着 macOS 并不知道你看不到刘海底下的内容，也就意味着当你的程序开的多的时候就会被挤到刘海里，然后你看不到它，看不到也就点不到。厉害吧，这就是优雅的 Apple 🤷‍♀️

那有什么办法可以让他们不长期驻守在那儿吗？比如说像 Windows 一样折叠起来？有的，你去搜一下就会看到很多人在推荐一个软件叫做 Bartender 的软件，它专职做这个工作。你开开心心把它下载下来发现：这个软件要 TMD 卖 $20，整整 TMD 20 dollars ！！！

> 这时候就有果粉要说了：「你不知道别乱说，明明还有开源免费的 [hidden bar](<https://github.com/dwarvesf/hidden>) 可以用呢」。啊确实，我就在用这个（反正不可能让我花 $20 隐藏一个图标） ，但是这东西不是苹果官方应该做的吗？？？

## Terminal

接下来是 Terminal，很多果粉在说 「macOS 是最适合程序员的系统」，那么我作为一个程序员就只配用这么烂的一个 Terminal 吗？Linux 桌面环境的 GNOME Terminal 或者 Konsole 和 Xfce Treminal 都比 macOS 官方提供的好用太多了。

不过好在 Terminal 没有太多收费的，很多都是开源项目，我这里推荐几个自己用过且好用的吧：

  * [Kitty Terminal](<https://sw.kovidgoyal.net/kitty/>) 高性能，使用配置文件，功能完备
  * [Alacritty](<https://alacritty.org/>) 高性能，使用配置文件，极简
  * [iTerm 2](<https://iterm2.com/>) 传统、功能完备、性能说得过去

# 0X05 万恶的 Command + Q

  * Windows 上切换不同窗口的快捷键是 Alt + Tab
  * Linux 上切换不同窗口的快捷键是 Alt + Tab
  * macOS 上切换窗口的快捷键是 Command + Tab

看似非常和谐（macOS 的 Command 其实就是 Windows 的 Alt），但是 macOS 中有一个类似于 Windows 中 Alt + F4 的强制退出快捷键：Command + Q。熟悉键盘键位的小伙伴应该已经发现了，Q 和 Tab 是 TMD 挨着的！是 TMD 挨着的！哪怕是你再熟悉键盘，盲打再怎么熟练，也有可能手抖按错一个键吧，那么极有可能出现你想切换窗口时直接把当前窗口 Kill 掉的情况。

我不理解为什么要把「关闭窗口」 和「切换窗口」两个快捷键搞的这么近，难道产品经理（或者说是乔布斯？）自己从来没有误触过吗？别的快捷键误触也就算了，但是把切换窗口误触成关闭窗口后果可是有点严重的啊。

> 我不信哪个长期用 macOS 的没出现过切换窗口不小心把窗口关掉的情况，绝对不可能 ❌

# 0X06 源自心底的傲慢

我们都知道在文件管理器的「网络」目录里可以看到同一个局域网内的其他设备，那么我们来看一下 macOS 是怎么表现「其他」设备的。

![macOS Finder Screenshot](https://cdn.just666.com/images/macos_finder_1.png)

没看清？放大看一下

![macOS Finder Screenshot](https://cdn.just666.com/images/macos_finder_2.jpeg)

是的你没看错，就是一个老式 CRT 显示器甚至还有些发黄了，运行着一个蓝屏了的 Windows 系统。那么 macOS 是如何展示自己的设备呢？

![macOS Finder Screenshot](https://cdn.just666.com/images/macos_finder_3.png)

是的，不仅区分了 Mac mini，MacBook 和 iMac，甚至区分了不同的 MacBook，每一个都是那么的现代且美观。苹果你真的有必要这样吗？
