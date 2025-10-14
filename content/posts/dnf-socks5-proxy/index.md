---
title: "Fedora 中 dnf 命令使用 Socks5 代理"
slug: "dnf-socks5-proxy"
date: "2018-03-14T15:59:00+0000"
lastmod: "2025-01-16T08:32:30+0000"
draft: false
tags:
  - "Linux"
  - "Proxy"
visibility: "public"
---
在Linux下安装软件通常会使用包管理工具自动处理依赖问题，在Fedora下一般使用`dnf`包管理工具。一般我们会给自己的源设置为国内的镜像源，比如

```
    https://mirrors.tuna.tsinghua.edu.cn
    https://mirrors.ustc.edu.cn
    https://mirrors.163.com
```

但是有的时候还是避免不了从国外源下载数据，这种情况下经常出现速度巨慢无比甚至会断开的情况。这种时候我们可以给`dnf`设置通过代理连接网络，这样一来下载速度就会快得多了。

`sudo vim /etc/dnf/dnf.conf`编辑`dnf`的配置文件，添加如下配置，保存后再执行`dnf`命令就可以使用代理的方式连接了。

```sh
    proxy=socks5://127.0.0.1:1080
    proxy_username=shawn
    proxy_password=shawn
```

需要注意的几点：

  1. 本示例只是针对我自己的电脑，如果你自己的Shadowsocks配置跟我的不同，请根据自己的配置自行修改（没有密码的可以不写后两项）
  2. 示例中使用了`socks4://`的协议，如果自己有其他方式的代理也可以使用，比如`http://`等
  3. 当不再使用的时候记得将配置注释掉，以防连接国内源也使用代理
