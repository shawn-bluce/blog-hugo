---
title: "在 Linux 桌面下活得舒服"
slug: "better-linux-desktop"
date: "2019-11-02T14:13:00+0000"
lastmod: "2025-01-16T07:32:03+0000"
draft: false
tags:
  - "Linux"
visibility: "public"
---
# 0X00 前言

这篇博客的目标读者：正在使用Linux桌面，打算长期继续使用下去的同学（这也就意味着熟悉Linux下的基础操作，理解Linux下的常见概念）。

这里有一个我之前写的“[在Linux桌面下存活](<https://blog.just666.com/2019/10/29/use-linux/>)”可以参考一下。

# 0X01 颜值就是战斗力

**Linux也不都是黑色背景白色字的命令行。**

首先要换的就是一套主题和图标，不论是KDE、Gnome还是Xfce都可以在对应的网站找到大量的主题，简单换过图标和主题之后再配合一张好看的壁纸，整个观感立马就好了不少。

> KDE: <https://store.kde.org/>
> Gnome: <https://www.gnome-look.org/>
> Xfce: <https://www.xfce-look.org/>

然后要换的就是字体了，[等宽字体](<https://zh.wikipedia.org/zh-hans/%E7%AD%89%E5%AE%BD%E5%AD%97%E4%BD%93>)我推荐这几个[Hack](<https://sourcefoundry.org/hack/>)、[Source Code Pro](<https://github.com/adobe-fonts/source-code-pro>)、[Cascadia Code](<https://github.com/microsoft/cascadia-code>)都很好看，适合在终端、IDE和编辑器里使用。

最后就是zsh主题，如果使用的是zsh的话推荐使用[powerlevel10k](<https://github.com/romkatv/powerlevel10k>)这个主题，是由[powerlevel9k](<https://github.com/Powerlevel9k/powerlevel9k>)发展而来的，但是速度比`powerlevel9k`快好多。

# 0X02 善用alias和bash函数

相信大家在初学Linux的时候都学过`alias`指令，没学过的话我简单介绍一句：“alias是给命令取别名的工具，例如执行`alias new_ls="ls -l"`过后再使用`new_ls`命令就和使用`ls -l`一样了”。

首先我们知道将`alias`命令直接写在shell里，在关闭重开shell之后就失效了（起码现在知道了）。所以我们要把`alias`写在`.bashrc`或者`.zshrc`中（依你使用的shell而定）。现在我的`.zshrc`中就有很多已经写好了的，下面给大家分项几个对大家都有用的

```sh
    # alias for simple command
    alias f='open .'    # 用图形文件管理器打开当前目录
    alias h='open ~'
    alias py='ipython3'
    alias py2='ipython2'
    alias du='/usr/bin/ncdu'    # ncdu是一个命令行里可视化查看磁盘（目录）占用率的工具
    alias cat='/usr/bin/bat'    # bat是一个带语法高亮、行号显示且能够上下滚动的cat加强版
    alias down='aria2c -x16 -j4'    # 使用aria2进行多线程下载
    alias me="cd $HOME/Workstadion/ && ls"
    alias connect_android="scrcpy --bit-rate 256M"  # 使用scrcpy连接到接入电脑的Android手机
    alias code='LANG="zh_CN.UTF-8" LANGUAGE="zh_CN.UTF-8" code'     # 用VSCode打开
    alias jwt="python3 $HOME/Workstadion/script/get_token.py jwt"   # 自己编写的方便获取开发环境jwt token的工具
    alias token="python3 $HOME/Workstadion/script/get_token.py token"
    alias open='LANG="zh_CN.UTF-8" xdg-open > /dev/null 2> /dev/null'   # 使用对应的工具打开文件

    # alias to source command   # 映射到原始工具
    alias _du='/usr/bin/du'
    alias _cat='/usr/bin/cat'

    # alias some command for network and proxy  # 使用代理做一些事情
    alias use_proxy="ALL_PROXY=socks5://127.0.0.1:1080"
    alias git_clone_with_proxy="ALL_PROXY=socks5://127.0.0.1:1080 git clone"
    alias yay_with_proxy="ALL_PROXY=socks5://127.0.0.1:1080 yay"
```

`alias`的功能还是比较单一，毕竟从“别名”这个名字上就看出来了。不过好在还可以写函数，写函数这就是我们程序员比较擅长的了。这里简单介绍（真的超简单的那种）一下`Bash`的函数（当然`zsh`也是兼容的）


    foo() {
        # xxxxx
    }


这就是一个函数的基本结构了，再配合参数：`$0 $1 $2 $3`这种就足够搞一些特别简单的函数了。我们一段命令如果前面固定后面是变动的那么可以方便得用`alias`比如：`docker-compose up -d web`和`docker-compose up -d test`这种就可以将前半部分搞成`alias`再拼起来用比如`alias dp-up-d="docker-compose up -d"`然后`dp-up-d web`就可以了。但是如果是`grep -Rn "hello" ~/articles`和`grep -Rn "world" ~/article`这种呢就比较麻烦，这时候就要用函数了。

```sh
    grep_this() {
        grep -Rn "$1" .
    }
```

这个函数就可以通过`grep_this hello`来执行`grep -Rn "hello" .`这个命令。其中的参数：`$0`和`$1`指的就是整个命令用空格分隔之后的下标，例如`$1`指的就是整条命令用空格分隔后，下标为1的值，这里的实例函数也就是指的"hello"了。

这里给大家复制几条我自己的配置，可能是大家都用的到的

```sh
    # v2er translate    # v2ex一老哥搞的词典api，直接用`v2 hello`就可以查到hello的中文（支持中英互转）
    v2() {
      declare q="$*"
      curl --user-agent curl "https://v2en.co/${q// /%20}"
    }

    # docker    # 使用`attach mysql`可以接入到含有mysql的行的docker容器的bash中（当有且仅有一个grep结果时才有效，有兴趣的老哥可以自行修改）
    attach() {
      docker exec -it `docker ps | grep $* | awk -F ' ' '{print $1}'` bash
    }

    attach_django() {
      docker exec -it `docker ps | grep $* | awk -F ' ' '{print $1}'` python manage.py shell
    }

    grep_this() {   # `grep_this test`从当前目录递归找下去，将所有含有test的文本文件的行都输出出来
      grep -Rn "$1" .
    }

    find_this() {   # `find_this test`递归当前目录找到路径含test的
      find . -name \*$1\*
    }
```

# 0X03 本地服务尽量Docker化

传统方式我们都是将本地开发所依赖的服务例如：MySQL和Redis这种直接部署一份。但是其实这种部署方式在开发环境是不太好的，强烈建议使用Docker部署。如果你同时要用3个MySQL版本，3个Redis版本，这只还要用5个Mongo版本，那怕不是要疯了对吧。

首先我们搞一个`docker`再搞一个`docker-compose`然后开始尝试部署环境（docker和docker-compose的安装这里就不说了）。

```yaml
    version: "3"
    services:

      mysql:
        image: mysql:latest
        container_name: dev_mysql
        network_mode: bridge
        restart: always
        volumes:
          - ./data/mysql:/var/lib/mysql
        ports:
          - "3306:3306"
        environment:
          - MYSQL_ROOT_PASSWORD=你的密码
        command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci


      redis:
        image: redis:latest
        container_name: dev_redis
        network_mode: bridge
        restart: always
        ports:
          - "6379:6379"

      mongo:
        image: mongo:latest
        container_name: dev_mongo
        network_mode: bridge
        restart: always
        ports:
          - "27017:27017"
        volumes:
          - ./data/mongo:/var/lib/mongo
```

我们将内容保存为`docker-compose.yml`，并放到一个空目录下开始一波复杂的操作：首先执行`docker-compose up -d`，然后等一会儿，等拉完这三个镜像，接下来。。。。。。就好了（是的，这就好了）。这样你就在本地部署了`MySQL+Redis+Mongo`并且都是最新的版本，如果需要不同的版本可以这个样子

```yaml
    version: "3"
    services:

      mysql:
        image: mysql:latest
        container_name: dev_mysql
        network_mode: bridge
        restart: always
        volumes:
          - ./data/mysql:/var/lib/mysql
        ports:
          - "3306:3306"
        environment:
          - MYSQL_ROOT_PASSWORD=你的密码
        command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

      mysql_old:
        image: mysql:5.7.17
        container_name: dev_mysql
        network_mode: bridge
        restart: always
        volumes:
          - ./data/mysql:/var/lib/mysql
        ports:
          - "3306:3306"
        environment:
          - MYSQL_ROOT_PASSWORD=你的密码
        command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

这时候用`docker-compose up -d mysql`和`docker-compose up -d mysql_old`就可以启动相应的版本了。

**注意**

> 这里展示只是改了版本，不过改了版本的话数据文件的映射一定要改，要不然两个不同版本的MySQL用同一套数据文件会出问题的这里展示只是用了最简单的配置，除了MySQL其他的服务甚至没有密码，即使是开发欢迎也要**注意数据安全** 如果需要更深度的自定义每个服务就需要取看对应镜像和服务的官方文档了

事实上我还用Docker部署过FTP方便同事的不同操作系统间快速传递文件，所以大大小小的服务只要想得到都可以用Docker来部署，不仅方便还能降低搞坏了的风险。

# 0X03 一个Py的小操作

现在有歌场景：你同事想从你电脑上搞个文件过去，该怎么搞？当然这有很多方法，但是有一个是非常简单快速的：找到你文件所在的目录`python3 -m http.server 2333`然后把`http://ip:2333`发给你的同事就好了。是的你没看错，用一行命令就启动了一个临时的web服务器。

这个操作应用场景也很多：你通过跳板机到服务器上导出了个数据，因为用的跳板机所以不能scp，那这时候`http.server`就是个好办法；甚至我还临时用这个命令给同事提供过[CXK打球](<https://github.com/kasuganosoras/cxk-ball>)的游戏呢哈哈哈。

# 0X04 zsh没插件就像喝酒没酒精

`zsh`作为一个所谓的“终极Shell”，支持的插件是非常非常多的。这里给大家看看我正在用的插件，配置文件是`~/.zshrc`：

```sh
    plugins=(
        z
        git
        fabric
        extract
        thefuck
        fzf-zsh
        git-open
        virtualenvwrapper
        colored-man-pages
        zsh-autosuggestions
        zsh-syntax-highlighting
    )
```

这里挑几个介绍一下

## z

我们在Linux下最常用的可能就是`cd`了，`z`配合`zsh`的原生功能可以让我们不再需要`cd`。比如我们经常去`~/Workstadion/too_young/simple`这个目录，那么以后就可以直接`z simple`跳过去了。其实是这样，你可以在终端直接用`z`来查看当前各个目录的权重，`z`命令就是从权重最高开始找，直到找到对应的目录并跳转过去。

另外我说的`zsh`原生功能是指：默认就是cd。比如我们在`zsh`中直接输入`/`就会跳到根目录，直接`..`就是上级，直接`~/Workstadion`就是对应的目录，省下了`cd`的过程

## fabric

如果你用`fabric`的话，在`fabfile.py`存在的目录下输入`fab `再tab就可以补全`fabric`中的命令

## extract

我们知道解压`zip`要用`unzip`，解压`rar`要用`unrar`，解压`tar.gz`要用`tar -zxvf`......知道是知道了，但是每次用起来还是有点麻烦，所以有了这个插件。不管你是啥`tar.gz/tar.xz/tar.bz2/zip/rar`，统统一个`x`全部解压。

```sh
    x qwer.zip
    x asdf.rar
    x zxcv.tar.gz
    x qwer.tar.xz
```

## thefuck

顾名思义，当你想吼`fxxk`的时候，这个插件就有用了。`thefuck`是一个python程序，要用`pip install thefuck --user`来安装。装好之后再配合上这个插件就可以实现神奇的效果。比如当你在命令行里输入了一长串命令`git commmt -m "feat: 新增了什么鬼的需求点"`，显然前面的`commit`拼错了，如果是以前的话就只有返回去改。这时候就很想大吼`fxxk`了，但是请淡定。双击一下你的ESC键，会发现你的命令已经被自动修正并写在命令行里了。

是的，这个`thefuck`就是自动修复写错的命令的工具。

## zsh-autosuggestions和zsh-syntax-highlighting

这两个都是需要手动安装的，可以去GitHub上搜一下，很简单。分别是实时提示命令的正确性和预先提示类似的历史命令的

## 一句话介绍其他的：

  1. git 是一堆现成的git的`alias`
  2. fzf-zsh 需要先安装`fzf`。在命令行里按下Ctrl+R，用新的逆向搜索替换原来不好用的老式搜索
  3. git-open 用浏览器打开当前git仓库：支持GitHub和GitLab
  4. colored-man-pages 彩色高亮的man

# 0X05 好像没啥了

一时半会也想不到有什么要补充的了，等什么时候想到新的了再搞吧。祝大家生活工作愉快~
