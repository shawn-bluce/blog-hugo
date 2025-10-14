---
title: "用好 Linux 之：软件推荐"
slug: "linux-software-recommend"
date: "2017-09-29T14:03:00+0000"
lastmod: "2025-01-17T02:09:44+0000"
draft: false
tags:
  - "Linux"
visibility: "public"
---
# 0X00 推荐一波Linux下的软件

Linux对于普通用户可能确实没有那么友好，但是对于计算机“专业”人士来说就好多了。我从接触Linux到现在也有个三两年了，而且用Linux桌面也有一段时间了。这段时间里也发现了不少好用的软件和工具，在这里整理一下也向大家推荐一波。这些工具有些是用来提升工作效率的，有些是用来娱乐的等等。。不过每一个都是我离不开的好工具。
非常重要的一点是，我推荐的这些软件除了为知笔记以外都是 **免费的** ，而且还有一大半是 **开源的** 。

# 0X01 zsh

Linux真正效率高的地方就在于Terminal，那我们就需要选一个好用的SHELL。目前绝大多数Linux发行版本都默认使用Bash，这是一个非常好用非常成熟的SHELL。但是有一个更好用的Shell叫做zsh，可以从官网安装也可以直接`apt install zsh`就装上了。这个shell一定要配合着`oh-my-zsh`来使用。[oh-my-zsh](<https://github.com/robbyrussell/oh-my-zsh>)可以让你的SHELL发挥最大的效率。

# 0X02 vim

vim 的好处就不多说了。在这里推荐一套开源的配置，在自己没有配置vim的情况下直接安装就好了。

```sh
    wget https://raw.githubusercontent.com/vince67/v7_config/master/vim.sh
    bash vim.sh
```

> 这个脚本官方只支持Mac和Debian系，如果是其他系需要自行修改一下这个脚本里的内容。
> 例如：如果是CentOS的话就将里面安装软件的`apt-get`修改成`yum`，安装就能顺利进行了。

这个安装好了之后vim常用的插件和配置就都搞定了，默认情况下写Python就可以拥有非常好的用户体验。具体用法简单看一下`~/.vimrc`就可以了解到了。

# 0X03 htop

在Linux命令行中查看系统资源比如CPU、内存、进程等要用到`top、free`等命令，现在一个`htop`就解决了问题，而且是彩色输出还支持鼠标点击排序。

![htop截图](https://raw.githubusercontent.com/shawn-bluce/shawn-bluce.github.io/enclosure/image/htop.png)

```sh
    sudo apt install htop
    sudo yum install htop
```

# 0X04 mycli

MySQL是我们常用的一款数据库了，有的时候需要连到数据库里查一些东西或是一些什么操作。通常我们会选用`mysql`命令来连接数据库，但是这个工具挺不好用的，所以才会出现了这么一款神器`mycli`。由于是用Python写的，还封装了pip，所以安装起来很简单，一条命令`pip install mycli`就搞定了。这个工具和`mysql`命令用法是完全一样的，他的特点就是支持自动补全和SQL高亮，而且输出默认是使用`less`展示的，可以直接用键盘上下滚动，不需要鼠标键盘乱换着用。

![mycli截图](https://raw.githubusercontent.com/shawn-bluce/shawn-bluce.github.io/enclosure/image/mycli.png)

# 0X05 WPS

[WPS](<http://linux.wps.cn/>)是在Linux下MS Office的最佳替代品了。虽然功能和易用性上不如MS Office，但是他免费，在Linux下没有广告，而且对MS Office的支持率还是很高的。通常只是写个简单的文档或者展示PPT什么的是没有问题的。不过WPS暂时还没有开源，倒是有一个开源的替代品LibreOffice，但是实在是不好用，有兴趣的同学可以加入LibreOffice的社区，帮助LibreOffice变得更好。
对了，WPS在Linux下的启动速度奇快，在我PCI-E Nvme的硬盘下每次都是秒开，比Office 2016要更快一筹。

# 0X06 Chrome/Firefox

这俩就没的说了，我就介绍一下我正在用的几个Chrom扩展吧

  1. Adblock Plus 去广告神器
  2. crxMouse Chrome Gestures 鼠标手势神器
  3. Draw.io Desktop 一个在线画流程图的ChromAPP
  4. Google翻译 可以设置为选中翻译，因为Google翻译没有被墙，所以很好用
  5. LastPass 一款免费的密码管理扩展，安全可靠方便易用
  6. Postman 一个用于快速模拟http请求的ChromeAPP
  7. Proxy SwitchOmega 一个用于设置浏览器代理的扩展，配合SS使用感觉良好
  8. Search by Image 以图搜图，开启后网页每张图右下角都会出现一个小Logo，点击Logo就可以用Google搜索这张图
  9. Tamepermonkey 一个JS脚本管理器，神器
9.1. AC-baidu 去百度广告
9.2. 百度广告清理 去百度广告可能需要两个才行
  10. Wappalyzer 开发者必备，可以显示出当前网页/网站所使用的技术和框架
  11. Vimium 用vim快捷键来操作浏览器，神器

# 0X07 为知笔记/蚂蚁笔记

在我所知道的范围内，Linux环境下体验还不错的笔记软件就这两款。给大家列个表对比一下 [为知笔记](<http://www.wiz.cn/>) 和 [蚂蚁笔记](<https://leanote.com/>)

特性 | 为知笔记 | 蚂蚁笔记
---|---|---
客户端开源 | 是 | 是
服务端开源 | 否 | 否
收费情况 | 免费体验100天，过后60一年 | 免费（不能同步）、50/年（支持同步，流量较少）、150/年（流量中等
一键变博客 | 不支持 | 支持
移动端体验 | 好 | 较差

总的来说，为知笔记每年要花60块钱，可以获得不错的使用体验；蚂蚁笔记每年要花150块钱才能获得不错的体验。不过由于蚂蚁笔记的服务端开源，所以可以将服务端部署在自己的服务器上以免费获得最完整的功能。主要就是看大家对哪点的需求更多了。

# 0X08 网易云音乐

网易云音乐就没必要过多介绍了。官方推出了deb包，几乎所有基于Debian的Linux都能一键安装。也有民间玩家制作了rpm包，没有用过不知道体验怎么样。还有民间高手在做的命令行版本的网易云音乐，功能已经相当完善了。目前有[Python版本](<https://github.com/darknessomi/musicbox>)的和[NodeJS版本](<https://github.com/Binaryify/NeteaseCloudMusicApi>)的，有兴趣的同学可以参与到开发过程中来。

# 0X09 Atom/vscode

这两个应该是目前为止图形界面下最好用的两款开源编辑器了。[Atom](<https://atom.io/>)是由Github制作的，[vscode](<https://code.visualstudio.com/>)是由Microsoft制作的。一个是拥有全世界最厉害程序员的平台，一个是全世界最厉害的软件公司之一。自然软件的质量没的说了，我个人也只是轻度使用，所以主要是看外观和心里偏向而已。比如我就比较喜欢用Atom。

# 0X0A Nylas

目前为止Linux下最有名的两款邮件客户端应该是雷鸟和EVO了，不过前段时间出来的这款[Nylas](<https://www.nylas.com/nylas-mail/>)特好用，我一直在用。最重要的一点是可以不用梯子就访问Gmail。不过目前遇到的唯一一个坑就是连接QQ企业邮箱的时候有问题，每次都会报错，但是又能收到信，很奇怪。给社区反馈过，也没得到答复。然而我还是强烈推荐这款软件，真的是好用。

# 0X0B Guake

这个名字对于大学生来说非常不友好，也不知道怎么这么好的软件怎么就取了个"挂科"的名字呢，哈哈。我们在用Linux的时候偶尔会需要输入一两行命令，或者是需要用htop长期看着自己的性能指标。那这时候每次都打开一个终端再输命令吗？当然不用。装一个`Guake`，给他设定一个快捷键，每次需要临时调出终端的时候只需要按一下快捷键，终端就会从屏幕顶部弹下来，还能设置失去焦点自动隐藏。这样用起来就很舒服了。

# 0X0C Virtualbox

这款软件想必也不需要介绍，是目前免费桌面虚拟机软件中最好的一款了。大量人在用破解的VMwareWorkstadion，虽然比VirtualBox好用一点，但是我们还是应该按版权来，不想花钱就用VirtualBox吧，况且也挺好用的。比如你在长期用Linux的环境下偶尔也需要用一下Windows，比如你长期用Fedora偶尔也要用一下Ubuntu，这就是[VirtualBox](<https://www.virtualbox.org/wiki/Downloads>)出场的时候了。

# 0X0D TeamViewer

平时在Windows下大家都用习惯了QQ的远程协助，那么到了Linux下应该怎么办呢？[Teamviewer](<https://www.teamviewer.com/zh/>)应该就是最好的选择了。其实Teamviewer在性能上是要比QQ的远程协助更强的。注意一点：这款软件对个人免费，但是要在公司里用的话是要花钱买的。

# 0X0E 深度终端

[深度终端](<https://www.deepin.org/2016/09/22/deepin-terminal-v2-0-released-all-can-be-done-in-terminal/>)是我目前在Linux里用过最好用的终端模拟器了。自带的主题很漂亮，对中文支持完美，而且还集成了SSH管理功能。

# 0X0F Shadowsocks-QT

翻越长城必备神器 [Shadowsocks-QT](<https://github.com/shadowsocks/shadowsocks-qt5>)。目前为止这东西是最好用的，但是不能像Windows和Mac中的那样直接支持PAC，需要浏览器的扩展来配合实现PAC功能。

# 0X10 Jetbrains

[Jetbrains](<https://www.jetbrains.com/>)他们的产品简直是厉害，如果是学生的话可以去申请全部软件的专业版。我用过的有WebStorm/PyCharm/IntellJ IDEA/DataGrip/Android Studio这五款，其中AndroidStudio免费，PyCharm和IDEA提供社区版。他们的软件最大的特点就是统一性强，你用过其中一款之后再去用其他的很快就能上手。而且都非常智能，这才是IDE应该有的样子。

# 0X11 Audacious

如果你有把音乐下载到本地再听的习惯，那么这款软件就特别适合。这款软件特别小，界面简洁，使用简单。是我用过Linux下最好用的离线播放器了。

# 0X12 Wireshark

一款可以用来分析网络流量包的软件，可以在官网下载最新的也可以在包管理中安装使用。这应该是开发者必备的工具之一了吧，有的时候为了分析一波流量或者看看自己的请求到底是不是有问题等等。。。当然，如果是大黑客的话还能有更炫酷的应用场景。

# 0X13 Dia

一款用来画图的开源软件，可以简单的画出流程图，数据库模型等等。。使用体验还是不错，如果只是轻度使用的话足够了。

# 0X14 Stellarium

如果你是一个理科生甚至以为天文爱好者，那你一定对宇宙的浩瀚很痴迷。这款软件能实时模拟星空的运动轨迹，还有各种行星、恒星的数据，官方中文。很值得一玩

# 0X015 VLC

应该是世界上最厉害的视频播放器之一了，毕竟是开源软件所以自定义程度非常强悍。是我在Linux中用过最好的视频播放器。
