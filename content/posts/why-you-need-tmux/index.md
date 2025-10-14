---
title: "你为什么需要会用 tmux"
slug: "why-you-need-tmux"
date: "2023-05-17T14:17:00+0000"
lastmod: "2025-03-04T08:55:19+0000"
draft: false
tags:
  - "Tmux"
  - "Terminal"
  - "Linux"
visibility: "public"
---
# 0X00 简单介绍

想必看到这篇博客的各位肯定会经常工作在 Terminal 中吧，而且对自己稍微好一些的人应该也都会配置一下自己的终端环境，比较常见的就是 Linux 下装个 terminator 或者 macOS 下装一个 iTerm2 这种软件，然后再用 zsh 配合不同的主题和插件完善自己的体验。而且真正用过一段时间终端的人肯定都会有那种一个窗口不够用的情况，那么你可能要用到终端模拟器（terminator/iTerm2）的 tab 功能了，每次都额外开一个新 tab 出来，或者上下左右开始分屏了。

一切都很顺利，直到你开始频繁的连接到远端的服务器上去，然后发现自己习以为常的分屏和 tab 全都没有了，每次想再开一个远端的 shell 时都需要在本地开一个分屏然后重新 ssh 重新输密码，需要 sudo 的话还可能需要再重复一下密码。一次两次还好，次数多了肯定就麻了，这时候就是 tmux 大展身手的时候了～

tmux 本身是一个终端复用器，可以做到的功能包括终端横向纵向的分屏、多 tab 切换等等。

# 0X01 基本用法

要用 tmux 首先要有 tmux 才行（废话文学），有些 Linux 发行版本预装了的，如果没有的话用对应的软件包管理器装一下就行了，非常小且没什么依赖。

在使用之前先要区分一下 tmux 中 `Server/Session/Window/Pane` 这四个比较重要的概念：

  * Server 服务，是最上级的，是整个 tmux 的后台服务，一般很少会直接操作它
  * Session 会话，是我们在终端敲下 `tmux` 之后随之启动的东西，类似于其他终端模拟器的一个窗口
  * Window 工作区，默认情况下新建一个 Session 就会带有一个 Window，类似于其他终端模拟器的 tab，也就是说一个 Session 可以创建很多个 Window
  * Pane 就是最小一级了，默认情况下一个 Window 就会带有一个 Pane，也就是说一个 Window 下面可以选择左右上下分很多个 Pane，类似下面这张图（这个是我自己配置过的，跟原生配置不同但是概念是一致的）

![SCR-20230517-txv](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2023/05/17/scr20230517txv.png)

> 图里看到的是一个 Session，我在其中创建了 4 个 window 并且在 window 4 上创建了 3 个 pane

然后打开自己的终端，输入 `tmux` 并回车就可以了，看起来和刚刚没什么区别。现在对着这个平平无奇的终端开始介绍一下具体的用法。

`tmux` 本身有非常多的快捷键，使用这些快捷键的前提是「进入 tmux 的命令模式」，默认按键是 `<Ctrl> + B` ，熟悉 vim 的人可能比较容易理解。因为默认配置下进入命令模式之后界面上并没有什么提示，所以不放心的话可以先敲两次 ESC 回到普通模式再按触发键。

要搞成上面我截图的这种（指的是用法不是外观）是很简单的，首先我们先按快捷键 `<Ctrl> + B` 进入命令模式，然后再按 `b` 就可以创建一个新的 window 了，连续创建 3 个之后大概会长成这样（截图是一个没有经过配置的 tmux 的样子）

![SCR-20230517-u9t](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2023/05/17/scr20230517u9t.png)

可以看到下面有四个 window 了，其中打了星号的就是当前正在操作的 window。想切换不同的 window 的话就是命令模式下直接输入数字即可。

![SCR-20230517-ufn](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2023/05/17/scr20230517ufn.png)

接下来在当前 window 下进行分屏，命令模式下 `"` 是左右分屏，`%` 是上下分屏。在一个 window 下有多个 pane 的时候可以使用命令模式下的方向键在多个 pane 之间切换，存在光标的就是当前正在激活的 pane（通过绿颜色的边框同样可以快速找到当前正在激活状态的 pane）。

# 0X02 修改配置

「新安装一个软件之后第一件事不是开始用，而是先点开设置看看有什么好玩的东西」的人应该不止我一个喔。

tmux 的配置文件是 `~/.tmux.conf` （其实还有其他会影响全局的，但是不建议改它所以这里并不打算告诉你），这里贴下我自己的一部分配置并在注释里简要说明一下，仅供参考。

```conf
    # bind new prefix  修改前缀按键（也就是命令快捷键）
    unbind-key C-b              # 取消绑定 Ctrl + b
    set-option -g prefix C-f    # 绑定 Ctrl + f
    bind-key C-f send-prefix

    # using vi mode
    setw -g mode-keys vi        # 使用 vi 模式
    bind-key -T copy-mode-vi 'v' send -X begin-selection        # vi 模式下的粘贴
    bind-key -T copy-mode-vi 'C-v' send -X rectangle-toggle
    bind-key -T copy-mode-vi 'y' send -X copy-selection         # vi 模式下的复制

    # plugins   # 一些插件，可以在 GitHub 上找到详细介绍
    set -g @plugin 'tmux-plugins/tpm'               # 插件管理器
    set -g @plugin 'tmux-plugins/tmux-sensible'
    set -g @plugin 'tmux-plugins/tmux-resurrect'
    set -g @plugin 'tmux-plugins/tmux-continuum'
    set -g @plugin 'tmux-plugins/tmux-yank'

    # status bar    # 截图下方右侧的状态栏
    set -g status-style bg='black',fg='white'
    set -g status-interval 1
    set -g status-left '#{?client_prefix,,}'
    set -g status-right '#[bg=default]#[fg=default]#(date "+%R ")#[bg=black]#[fg=brightgreen]#[bg=brightgreen]#[fg=black]Shawn#[bg=brightgreen]#[fg=black]'   # 字体原因可能现实不全，但是可供参考

    # window bar    # 截图下方左侧的 window bar
    set -g window-status-current-format "#[bg=brightgreen]#[fg=black]#[bg=brightgreen]#[fg=black]#I:#W#[bg=black]#[fg=brightgreen]"   # 同上字体原因
    set -g window-status-format "#[bg=default]#[fg=default]#I:#W"

    # pane border   # pane 之间的分割线配置
    set -g pane-border-style fg='gray'
    set -g pane-active-border-style fg='brightgreen'

    # window and pane index     # 违背祖宗之法的从 1 开始编号
    set -g base-index 1
    set -g pane-base-index 1


    # hotkeys   # 配置一些快捷键
    bind h select-pane -L   # 继承自 vim 的 h 为切换到左侧 pane
    bind j select-pane -D   # 同上继承自 vim
    bind k select-pane -U   # 同上继承自 vim
    bind l select-pane -R   # 同上继承自 vim
    bind y resize-pane -L 5 # 选取 hjkl 上面四个按键，用于上下左右拉伸 pane
    bind u resize-pane -D 5
    bind i resize-pane -U 5
    bind o resize-pane -R 5
    bind r source-file ~/.tmux.conf \; display-message "Config reloaded.." # 修改配置文件后方便重载配置

    set -g display-panes-time 3000
    set-option -g mouse on  # 启用鼠标操控

    # split pane
    unbind '"'  # 弃用双引号分割
    unbind %    # 弃用百分号分割
    bind - splitw -v -c '#{pane_current_path}'      # 将 - 绑定为上下分屏（图像记忆，像是横着的一刀）
    bind \\  splitw -h -c '#{pane_current_path}'    # 将 \ 绑定为左右分屏（同上）


    # run TmuxPluginManager     插件管理器
    run '~/.tmux/plugins/tpm/tpm'
```

> ⚠️ tmux 就像 vim 一样，你当然可以在自己的电脑上疯狂配置，但是请牢记：首先不论如何都要记得默认配置下的基本用法，否则当你远程到服务器上的时候必定会一脸懵逼；其次千万不要修改服务器上的操作相关的配置，否则你的同事可能会提刀来见你☠️

# 0X03 命令速查

> 默认配置

系统指令：

  * ? 帮助文档
  * d 断开会话
  * D 选一个 session 断开
  * C-z 挂起当前 session
  * r reload 当前 session
  * s 显示 session 列表并切换
  * : 进入系统 shel
  * [ 进入复制模式，按 q 退出
  * ] 粘贴复制模式的文本

window 指令

  * c 新建
  * & 关闭当前
  * 0~9 切换
  * p 上一个
  * n 下一个
  * w 打开列表用于切换
  * ‘ 重命名当前 window
  * . 修改当前编号
  * f 快速定位（匹配名称

Pane 指令

  * ” 左右分屏
  * % 上下分屏
  * x 关闭当前 pane
  * z 临时全屏，再按恢复
  * ! 将 pane 挪出当前 window
  * ; 切换到上一个 pane
  * { 向前置换
  * } 向后置换
  * Alt + ↑↓←→ 调整当前 pane 边缘
  * t 显示时间
  * o 切换 pane
  * q 显示 pane 的 index，快速输入 index 可切换

# 0X04 更好的资料

是的没错，[阮一峰大佬的文章](<https://www.ruanyifeng.com/blog/2019/10/tmux.html>)当然比我写的更好很多很多倍😄
