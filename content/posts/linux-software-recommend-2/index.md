---
title: "Linux 下日常使用软件推荐"
slug: "linux-software-recommend-2"
date: "2018-08-12T05:05:00+0000"
lastmod: "2025-01-17T02:17:16+0000"
draft: false
tags:
  - "Life"
  - "Linux"
visibility: "public"
---
本来准备总结一下Linux桌面系统使用一年以来的一些感受，以及为什么选择Linux作为桌面系统工作学习的，但是构思了半个小时也没能想到要写些什么。所以还是来推荐一下一年以来在Linux桌面平台下的软件体验和推荐吧。由于我这一年多以来一直使用的是Fedora Workstadion，所以并不能保证这些软件能在其他平台下的体验与我一致，不过一般来说都是没有问题的呢。

# 0X00 vim + spacevim

首先说明我不认为Vim比IDE写代码更好用，但是我仍然在使用vim。一个原因是觉得使用盗版IDE有些不太道德，另一个也是想要提升一下自己的代码水平，毕竟vim给出的提示会更少一些。IDE用户可以在IDE上安装vim的操作插件，毕竟vim的操作方式还是能很大程度上提升效率的。

[SpaceVim](<https://github.com/SpaceVim/SpaceVim>) 呢是vim的一个插件打包配置集成之后的一个版本。由于它集成了大量精选插件和配置，所以用起来很舒服，就把他当成一个高级的文本编辑器来用就好的。spacevim还有官方的中文文档可以查阅，虽然学习成本相对高一些，但是带来效率和爽快感的提升可不是一点点的。

# 0X01 ulaunch

[ulaunch](<https://github.com/Ulauncher/Ulauncher>)是一个快速启动器，按下快捷键后屏幕中间会出现一个搜索框，可以快捷打开软件和文件。配合一些插件可以实现在搜索框中翻译等工作，而且软件和插件都是Python写的可以轻松制作自己的插件。

# 0X02 plank

plank是一个简单轻便的dock栏，在Fedora的官方仓库里就有的，直接使用包管理器就能一键安装。

# 0X03 audacity

audacity是一个linux下的本地音乐播放器，简单好用，没有多余的复杂功能，推荐一波。

# 0X04 vlc

vlc应该是Linux下最好的视频播放器之一了，各种格式的视频音频都不是问题。

# 0X04 filezilla

这是一款老牌的FTP工具，虽然大多数时候文件管理器已经可以满足我们对FTP的一些基本需求了，但是偶尔还是会有一些满足不了的东西，这时候filezilla就能发挥作用了，留一个有备无患嘛。

# 0X05 electronic wechat

鹅厂没有Linux下的微信，不过幸好有人制作了这个基于网页版微信的微信客户端，使用体验良好，推荐！

# 0X06 electronic ssr

shadowsocksR没的解释。对应的还有一个Shadowsocks-QT5也很棒，不过不支持SSR罢了，看需求选择。

# 0X07 zsh + ohmyzsh

命令行组合，再搭配上ohmyzsh的一些插件，不仅终端非常漂亮效率还非常高。说到zsh就不得不说fish，相对来说zsh是和bash高度兼容的，虽然比fish慢了一点，但我更喜欢与bash的高度兼容，根据喜好选择啦。

# 0X08 jq

jq命令是用来格式化json输出的。比如你的`curl -XGET xxxx`和`cat xxx.json`的输出结果，通常都是没有格式化的，如果你安装了jq的话就可以`cat xxx.json | jq`把输出变成格式化的，而且带有高亮。注意哦，这个包在fedora中就直接叫做`jq`所以只需要`dnf install jq`就可以了，其他的发行版本自己找一下哈。

# 0X09 ag

ag是又一个超短超好用的命令。有这样一个场景，你在代码库的根目录下，想要找几百几千个源码文件中的`pdb`该怎么办呢？熟悉linux的很容易写出`grep -Rn "pdb" .`，从而找到所有包含pdb的行。ag就是用来替换这条较长且经常使用的命令的，你只需要用`ag pdb`就可以实现同样的效果，而且输出结果还比grep的更美观更直接。

> 在fedora中这个包名是`the_silver_searcher`。

# 0X0A vnote

在github中搜索可以找到一个名为Vnote的项目，这个是我用过Linux下最好用的离线笔记软件了。支持加密、markdown、公式渲染、流程图渲染、多笔记本、多层目录等等特性。而且只需要跟自己的同步云盘配合一下就直接变成了一个云笔记，完美。

# 0X0B 结尾

Linux下的好软件并非只有这些，这些只是我在日常使用中发现的好用的工具。其中人人皆知的就没有再重复写进来，只写了一些不是很知名的或是奇怪用法的软件。如果各位有什么推荐的工具可以留言哈。
