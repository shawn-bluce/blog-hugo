---
title: "关于 sudo 命令也许你需要知道的"
slug: "about-sudo"
date: "2023-05-24T14:00:00+0000"
lastmod: "2025-03-04T08:55:34+0000"
draft: false
tags:
  - "Linux"
  - "Permission"
  - "sudo"
visibility: "public"
---
# 0X00 介绍

不管对 sudo 的了解具体有多少，至少应该都用过 sudo 命令来临时将自己的非 root 用户提权至 root 了吧。不过 sudo 当然不只是将用户变成 root 的这么一个简单工具了，虽然它确实是将用户临时变更为 root ，但是页还是有不少其他更加细致的配置与选项。

> sudo 是 Linux 中的一个命令，用于以管理员身份执行命令。它允许普通用户在不切换到 root 用户的情况下执行需要特权的操作，从而提高了系统的安全性和可管理性。sudo 可以通过配置文件进行自定义，以控制哪些用户可以以何种方式执行哪些命令。同时，sudo 还可以记录用户的操作日志，以便系统管理员进行审计和监控。

还有一个需要注意的就是，sudo 本质上是一个应用，并非最基础系统的一部分，它比 Unix 晚诞生了有近 10 年。这也就意味着并不是所有的 Linux 发行版本都会自带这个程序（比如 minimal 模式安装的 Ubuntu Server 就是没有的），如果遇到这种情况还是需要自己手动安装一下，不过这种多人共用的操作系统中没有 sudo 的情况还是很罕见的。

# 0X01 修改配置

## 说在前面

sudo 的配置文件是 `/etc/sudoer`，虽然它是一个纯文本但是这里要提醒一下 **不要使用任何文本编辑器直接打开并编辑这个文件** 。因为一旦这个文件被改出问题来了，那可就麻烦大了。在一个没有给 root 用户手动设置密码的操作系统中，用自己的用户把 sudo 的配置文件改崩了，这时候想再 `sudo vim /etc/sudoer` 的时候极有可能因为配置文件蹦了导致 sudo 不能正常工作，这也就意味着可能你甚至所有用户都永久失去了成为 root 的机会，任何需要 root 的操作就都没办法操作了。

那应该怎么改这个文件呢？答案是用 `visudo` 这个命令。不要慌，即使你不会用/用不好 vi 也没关系，这个工具的名字虽然带有 vi 但其实是根据你环境变量的 `$EITOR` 来决定的，你也可以用 `EDITOR=/bin/nano visudo` 来临时用 nano 进行编辑。

那用 `visudo` 有什么好处呢？主要是下面两点

  1. visudo **带锁** ，当有一个用户正在编辑配置文件的时候你用了这个命令，你就会被通知并且没有办法修改该配置文件，直到对方保存退出了你才可以打开文件进行编辑。这样一来保障了多人同时编辑带来的问题。
  2. visudo 有**语法检查** ，也就是说每当你保存配置文件的时候都会进行一次基础的语法检查，防止该文件出现语法错误。如果单行配置写的不对，可能只会影响到对应的那行配置；但是如果语法有问题则会导致整个配置文件出错，问题就大了

最后关于如何编辑配置文件还有一个需要注意的是：使用 `visudo` 编辑配置文件后，保存并不会生效，要退出才会生效。这里可以注意观察一下，用 `visudo` 打开文件后直接保存，vi 的下面会提示将配置文件保存到 `/etc/sudoers.tmp` 了，只有当退出之后才会将配置写入到 `/etc/sudoers`

![sudoers_config_file](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2023/05/24/sudoersconfigfile.png)

> 一个 Linux 系统上的小 Tips：`man`命令不仅可以看某个命令的文档，输入 `man /etc/sudoers` 试试吧～

## 授权配置

授权配置就是 sudo 的核心功能了，首先配置每行就是一条，格式是这个样子


    USER/GROUP HOST=(USER[:GROUP]) [NOPASSWD:] COMMANDS


  * `USER/GROUP` 首先是本行配置对应的用户或者用户组，如果是组的话需要在前面加上`%`
  * `HOST` 表示允许从那些机器登陆的用户使用 sudo，通常的用法是 ALL 表示任何终端和机器
  * `(USER[:GROUP])` 表示使用 sudo 可以切换的用户/组，通常的用法是 ALL，也就是可以切换到任意用户
  * `NOPASSWD:` 表示使用 sudo 时不需要密码
  * `COMMANDS` 自然就表示允许执行的命令了，ALL 则表示允许执行所有命令

其中用方括号括起来的部分是可选内容，可以不出现在配置中。

下面是一些常用的配置示例

```
    # 允许 sudo 组执行所有命令
    %sudo ALL=(ALL:ALL) ALL

    # 允许用户执行所有命令，且无需输入密码
    USERNAME ALL=(ALL) NOPASSWD: ALL

    # 仅允许用户执行 echo, ls 命令
    USERNAME ALL=(ALL) NOPASSWD: /bin/echo,/bin/ls

    # 运行本机的用户执行关机命令
    USERNAME localhost=/sbin/shutdown -h now

    # 允许用户执行 /usr/sbin/ 下的所有命令，除了 /usr/sbin/useradd
    USERNAME ALL=(root) /usr/sbin/,!/usr/sbin/useradd

    # 允许用户以另一个指定用户的身份运行命令，且允许切换到另外一个指定的用户
    USERNAME ALL=(server) NOPASSWD: ALL, (root) NOPASSWD: /bin/su - server
```


## Defaults 配置

```
    # 指定用户尝试输入密码的次数，默认值为3
    Defaults passwd_tries=5

    # 设置密码超时时间
    # 其中 -1 是立即超时，页就意味着每次 sudo 都要重新输入密码
    # 0 是永不过时，登陆后输入一次密码就可以一直用 【不建议】
    # 5/10/12 其他数字的单位是分钟，默认就是 5 分钟
    Defaults timestamp_timeout=-1

    # 默认 sudo 询问用户自己的密码，添加 targetpw 或 rootpw 配置可以让 sudo 询问 root 密码
    Defaults targetpw

    # 保持当前用户的环境变量
    Defaults env_keep += "LANG LC_ADDRESS LC_CTYPE COLORS DISPLAY HOSTNAME EDITOR"
    Defaults env_keep += "ftp_proxy http_proxy https_proxy no_proxy"

    # 安置一个安全的 PATH 环境变量
    Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
```

# 0X02 基本原理

要说 sudo 的原理（并不能很深入）得先从它的工作流程说起

  1. `sudo` 会读取和解析 `/etc/sudoers` 文件，查找调用命令的用户及其权限
  2. 然后提示调用该命令的用户输入密码 (通常是用户密码，但也可能是目标用户的密码，或者也可以通过 NOPASSWD 标志来跳过密码验证)
  3. 之后，sudo 创建一个子进程，调用 `setuid()` 来切换到目标用户
  4. 接着，它会在上述子进程中执行参数给定的 shell 或命令

![linux_sudo_permission](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2023/05/24/linuxsudopermission.png)

首先我们看 `sudo` 这个命令位于 `/usr/bin/sudo`，查看它的权限和所属人发现这个文件属于 `root` 用户并且权限是 `--s--x--x`，这里的 `s` 位表示的是 `setuid`。关于 setuid 可以看[我的这篇博客中关于 suid 的介绍](<__GHOST_URL__/2021/06/02/linux-unbasic-permission>)。简单的说就是任何人执行这个程序的时候都会临时将自己的用户转换为 root，最后在 root 用户的权限体系下再进行配置文件的读取与校验。

# 0X03 参考资料

  * [深入理解 sudo 与 su 之间的区别](<https://linux.cn/article-8404-1.html>)
  * [关于 sudo 命令的一些配置和使用技巧](<https://kuanghy.github.io/2019/11/17/linux-sudo>)
