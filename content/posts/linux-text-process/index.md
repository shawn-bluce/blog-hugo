---
title: "Linux 文本三剑客 grep/awk/sed 入门手册"
slug: "linux-text-process"
date: "2024-03-21T13:29:00+0000"
lastmod: "2025-01-16T09:35:27+0000"
draft: false
tags:
  - "Linux"
visibility: "public"
---
# 0X00 前言

不知道为什么，当三个好用的工具在一起的时候就会被称作：三剑客；四个好用的工具在一起的时候就会被叫做四大天王 🤔。

算了，这不重要。

这篇文章的目的是带不了解这三个工具的朋友们简单上手使用它们，默认各位是掌握了 Linux 的基本用法的，其中也会出现有关正则的内容。如果你不懂正则的话建议跳过正则的部分，并且看完这篇文章马上就去学。另外，不要因为正则看起来有点像通配符就按通配符的操作进行下去。

# 0X01 grep

首先这三个工具中最常用的应该就是 `grep` 了，它用于从文件中搜索你感兴趣的内容。例如下面的例子就可以输出 `/etc/passwd` 文件中包含 `root` 的行

```sh
    grep root /etc/passwd

    >> output
    root:x:0:0::/root:/bin/bash
```

也可以接多个文件，这样输出的时候就会以文件名开头了

```sh
    grep root /etc/passwd /etc/group

    >> output
    /etc/passwd:root:x:0:0::/root:/bin/bash
    /etc/group:root:x:0:root
```

下面介绍几个参数：

  * `-i` 忽略大小写
  * `-v` 显示不匹配的行（取反）
  * `-n` 增加行号显示
  * `-c` 显示总共多少行，而非具体内容
  * `-r` 递归查找所有文件
  * `-A` 也就是 after，即显示匹配行和它后面的 n 行
  * `-B` 也就是 before，显示匹配行和它后面的 n 行
  * `-C` 相当于 `-A` 和 `-B` 一起用，显示匹配行和它前后各 n 行

简单列举一下使用方法

```sh
    # 忽略大小写
    grep -i ROOT /etc/passwd # 可以匹配到 root 行

    # 取反
    grep -v root /etc/passwd # 匹配到非 root 行

    # 递归查找
    grep -rn root /etc # 查找 /etc 目录下所有文件，输出所有带有 root 的行
```

grep 命令默认情况下就可以使用正则表达式，下面的命令就可以匹配到 root 行

```sh
    grep "r..t" /etc/passwd
```

grep 很多时候是跟在管道符号后面的，例如 `curl -XGET http://xxx.xxx/info/ | grep KEYWORD` 这种。注意跟在管道符号后面时，grep 只能处理来自标准输入的内容，如果还需要处理标准错误的话需要手动将其重定向到标准输出中。

```sh
    cat /etc/passwd | grep root
```

# 0X02 awk

相比于 grep 处理的是行数据，awk 则主要用于处理列数据。grep 可以从 `/etc/passwd` 中找到包含 root 的这一行，但是如果你想找所有用户他们用的 shell 就不容易了。

下面这个命令就可以在 `/etc/passwd` 中找到每个用户用的是哪个 shell 了。我们观察 `/etc/passwd` 这个文件可以看到里面每行都有多个字段，并且用 `:` 分割，第 7 列就是我们要的 shell。所以得到下面这个命令。

```sh
    cat /etc/passwd | awk -F ':' '{print $7}'

    >> output
    /bin/bash
    /usr/bin/nologin
    /usr/bin/nologin
    ..............
    /usr/bin/nologin
    /usr/bin/nologin
    /usr/bin/git-shell
```

先来分析一下这个命令，从 `awk -F ':' '{print $7}'` 开始。其中 `-F` 参数就是指定一个「分隔符」，后面的 `:` 就是分隔符本符，最后一个参数 `{print $7}` 表示输出第 7 列。其中 `{print $INDEX}` 可以理解成是一个固定的语法，也可以用 `{print $1,$7}` 的方式输出第 1 和第 7 列，且用逗号分割。

程序中第一列不是 `$0` 虽然有些诡异，但它也没有被弃用，可以试试 `{print $0}` 是什么作用。

> 这里的分隔符也是可以用正则的，只是一般来说这里用正则的时候比较少见。

# 0X03 sed

sed 的定义是一个「流编辑器」，如果理解不了就把他当成一个连 vim 那种界面都没有的编辑器好了。

**我们在开始之前先把`/etc/passwd` 复制一份到 `/data/passwd`，防止一会儿误伤操作系统**

如果你用过 vim 的话，上手 sed 应该是比较容易的。我们先来看一个例子

```sh
    sed "s/root/Administrator/" /data/passwd
```

这个例子就是将 `/data/passwd` 中的 root 换成 Administrator（好熟悉的用户名 🤣）。但是它只会修改每行的第一个 root，你可以观察一下你的输出是不是这样。如果你想替换的是所有的呢？知道的朋友肯定知道了：「只需要在替换命令后追加一个`g`」。是的，这就是典型的 vim 用法了。改过之后的命令应该是 `sed "s/root/Administrator/g" /data/passwd`。

这时候你可能发现好像你每次改的内容都回显到标准输出了，并没有对文件生效。但是也不要心急去用重定向（你不信邪也可以试试，反正文件复制出来了没有什么风险），想让修改直接对文件生效需要给 sed 命令加上 `-i` 参数，最后的成品如下，这样一来就可以将文件中的 `root` 全数替换成 `Administrator` 了。

```sh
    sed -i "s/root/Administrator/g" /data/passwd
```

如果你想删除某行，可以用 `sed "/root/d" /data/passwd` 的方式去删除。

> 如果你不信邪去试了重定向，会发现文件变成空白的了。这是因为当你的命令中出现重定向时，会优先准备重定向，也就是说当 sed 运行起来的时候该文件已经是等待输入的空白文件了。

> 也许你平时对 sed 并不感冒，但是当你需要处理一个很大的文本文件的时候，相信我，你一定会想起它的好的。

# 0X04 最后

通常来说这些工具会和 `cat`、`tail` 等工具一起用，通过管道将他们联系起来。

有关组合技的用法就太多了，CLI 最大的魅力可能就在于组合，不过还是需要各位在工作和学习中慢慢摸索～
