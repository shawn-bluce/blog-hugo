---
title: "Linux中诡异的权限（奇怪的权限增加了）"
slug: "linux-permission-simple"
date: "2020-04-23T15:26:00+0000"
lastmod: "2025-01-16T09:32:31+0000"
draft: false
tags:
  - "Linux"
  - "Permission"
visibility: "public"
---
# 0X00 前言

> Linux诡异的权限是怎么回事呢？Linux相信大家都很熟悉， 但是诡异的权限是怎么回事呢？下面就让小编带大家一起了解吧。
>
> Linux诡异的权限，其实就是诡异的权限了。那么Linux为什么会诡异的权限，相信大家都很好奇是怎么回事。大家可能会感到很惊讶，Linux怎么会诡异的权限呢？但事实就是这样，小编也感到非常惊讶。 那么这就是关于Linux诡异的权限的事情了，大家有没有觉得很神奇呢？
>
> 看了今天的内容，大家有什么想法呢？欢迎在评论区告诉小编一起讨论哦。

说正事说正事儿。说起Linux权限大家肯定："这我知道啊，不就是rwx吗，r是读、w是写、x是执行。就这？"当然不只是这个，不过我们还是要从最基础的开始说起来。

# 0X01 基础权限部分

首先最基础的权限就是 `rwx` 这种，三组权限针对:所 属用户、所属用户的组、其他用户，每组3位(对应 二进制位)。正因为对应二进制位所以`rwx`就是三个 二进制位均为1的7;`r-x`就是对应的101也就是5;`r-` 就是100也就是4

![Linux基础权限](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge43ddofakj315g0iu16s.jpg)

最基础的rwx权限就不多说了，说一个不是所有人都 知道的，看下面这张截图:这个叫做linux.pdf的文 件，这个文件的权限是777，但是当我们试图删除它的时候，发现完全不能行，那是为什么呢？我们很自然的认为对一个文件有rwx的权限就是有所有权限了，其实这么理解问题不大。但是考虑一个问题，删除一个 目录里的文件，实际上是不是在对这个目录进行w操 作呢?

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge43jtgw7xj30py06yq73.jpg)

返回来再看这个目录的权限就明白了。是的，基于Linux中"万物皆文件"的思路，可以知道目录其实也是文件，所以删除目录里的文件就是在修改这个目录，进而得到结论：删除文件是需要拥有对文件所在目录的w权限才行的。

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge43gladatj30v605agt8.jpg)

# 0X02 ACL

现在再来看另一个问题:我们看这个叫`macOS.txt`的文件，又是一个777权限的文件。按照上面提到的内容，我们就算不能删了它起码也能给它写成空文件是吧，因为毕竟有w权限。但是你真的有这个文件的w权 限吗?当你尝试给这个文件写入内容的时候直接就报错了，完全没有权限。是的，也许机智的你注意到了，问题就出现在 `-rwxrwxrwx+` 中最后的`+`那里。

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge43o1s5o6j30za08c7fp.jpg)

实际上是因为Linux上还有一个叫做ACL的机制。系统用户肯定不能单纯通过用户和组来完成的，正是通过 ACL的这个机制可以在传统的rwx权限的基础上进行扩 展。可以使用ACL在rwx之外给单独一个/多个用户/组 指定权限。比如下面这种用法:`setfacl -m u:shawn:--- macOS.txt`拆分开看这个命令，第一个参数 -m指 的是(modify)，后面的`u:shawn:---`就是说 `用户: shawn:三无权限` 。再使用getfacl看一下文件具体权限，可以看到，所属人和组都是root，用户、组和其 他人的权限都是777，但是只有一个`user:shawn:---`，这个就意味着只有这个用户是没有权限的。

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge43oxve6sj30yq0cy4ai.jpg)

# 0X03 隐藏属性

我们来看一下这个DELETE_ME的文件，我们仍然还是有777的权限，也没有通过ACL限制单个用户的权限，而且当前目录我也有w权限。那我们来尝试删除或者重写一下内容好了，发现还是还是还是还是没有权限。。。。那这回又是为什么呢？

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge43pp3dfuj310i0pm1kx.jpg)

是的，又是一个奇怪的东⻄:`attr`我们可以使用`lsattr filename`来查看当前文件的隐藏属性，有很多，这里可以看到的是i和e。其中e是系统底层的自带的， 有兴趣的话可以自己查阅一下资料。刚刚文件删不掉 改不了是因为这个i。这个i的功能就是: **immutable 不可改变的** 。所以我们不能删除，也不能修改内容。 使用`chattr -I filename`就可以将这个标记删除了，然 后这个文件也就可以改动了。如果需要增加这个标记 的话是`chattr +I filename`也就是说用加减号来控制隐 藏的标记。也可以使用等号`chattr =I filename`的方式来直接重写所有的标记。

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge43qbpmjij30zw08cn6h.jpg)

关于这个attr的所有参数内容[都在这里](<https://linux.die.net/man/1/chattr>)

# 0X04 奇怪的权限知识增加了

到此为止这些隐藏在最基础的rwx权限之外的奇怪的权限（并不全，其实还有其他的）就说完了，希望大家能有所收获～

备注1:有些Linux发行版本没有默认附带这个，我展 示是用的Fedora所以要通过dnf装一下这个功能。

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge43t8b5fvj32hc0quu0x.jpg)

备注2:除了rwx的基础权限部分以外，ACL和attr两 部分内容是仅适用于Linux发行版本的，macOS并不 适用，如果需要学习或者测试的话要搞一台Linux才 行。

备注3:ACL和attr的相关资料:

  1. <https://man.linuxde.net/setfacl>
  2. <https://man.linuxde.net/lsattr>
  3. <https://man.linuxde.net/chattr>
  4. <https://linux.die.net/man/1/chattr>
