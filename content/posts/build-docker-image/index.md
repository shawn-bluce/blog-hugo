---
title: "正确构建 Docker 镜像"
slug: "build-docker-image"
date: "2019-09-17T13:59:00+0000"
lastmod: "2025-01-16T08:33:16+0000"
draft: false
tags:
  - "Docker"
visibility: "public"
---
# 0X00 最常见的两种构建方式

构建Docker镜像的方式并不多，最常用的也就只有：编写Dockerfile和使用docker commit这两种。既然方式分为两种那么肯定是有区别的（废话），那我们来看看吧。

首先来介绍一下这两种构建方式，假设打算使用docker部署我们的服务，那么我们来使用两种方式来构建一下这个镜像吧。

# 0X01 docker commit

像我们这种新手平时用的比较多的应该就是`docker commit xxxxx hub.xxx.xxx/xxx:xxx`这种方式了，我们称之为`docker commit`。这种方式比较好操作，比较好理解，操作也比较容易。如果掌握了`git`的工作流程，那么使用`docker commit`方式来构建镜像简直是小菜一碟。

  1. 我们搞一个基础镜像比如`fedora`，那我们把它搞下来：`docker pull fedora`
  2. 运行并进入到容器里`docker run --name our_container -it fedora /bin/bash`，此时shell已经接入到容器里了
  3. 我们来安装吧 `dnf install python3`然后`pip3 install Django` 这样就装好了，可以退出docker里的shell了
  4. `docker commit -m "安装了Python和Django" our_container our_image:latest`这样就将刚刚的容器打包成名为`our_image`的镜像了
  5. 现在使用`docker images`就可以看到刚刚打包的`our_image`了
  6. 后面如果还需要更新这个镜像那么就可以继续运行这个`docker run --name our_container -it our_image:latest`然后更新完了再`docker commit -m "balabalabala" our_container our_image:latest`打包一层新的上去就好了

# 0X02 Dockerfile

另一种常见构建镜像的方式是编写`Dockerfile`文件，通过Dockerfile构建一个新的镜像。

  1. 创建一个名为`Dockerfile`的文件，并写入以下内容
  2. 在当前目录执行`docker build . -t our_image:latest`就可以了


```dockerfile
    FROM fedora
    RUN dnf install python3
    RUN pip3 install Django
```

# 0X03 对比两种方法

两者的对比从下面几个方面来进行

## docker hub

这两种方法其中`docker commit`方法通常需要使用私有`docker hub`，而`Dockerfile`则可以不使用。因为`docker commit`完成的镜像如果要分享给其他人的话最方便的途径就是走`docker hub`（自己构建完push到hub上，其他人再pull下来），而`Dockerfile`只要保证使用的是公共的基础镜像那么所有人就都可以直接构建出目标镜像。

> 之所以说是最方便，是因为还有一种`docker save`的方式可以打包镜像成`xxx.tar`，别人再`docker load`。这种方式虽然能用但是很麻烦，也不是常规操作

> docker 镜像当然可以上传到公共的hub上，但是公司的商业内容怕是不允许呦

## 使用难度

`docker commit`操作简单，每次更新镜像时接入到容器里一波操作最后`docker commit xxxxx`封装一下再使用`docker push`将镜像推到hub就可以了。基本所有人都可以很快搞清楚流程并且开始上手使用。

`Dockerfile`就显得难了不少，构建复杂一点的镜像往往`Dockerfile`就几十上百行了，再算上里面十多种`Dockerfile`语法，并不是所有人都能很快上手的。

## 镜像层数

众所周知**Docker镜像的每次commit都会在文件系统上新增一层** （反正现在你肯定知道了），而层数的增加意味着“镜像体积膨胀”。如果决定使用`docker commit`的方式维护镜像，那么不免后期会大量通过commit来更新镜像从而导致镜像层数过多，体积过大。

而`Dockerfile`中的每个指令也都会为镜像摞上一层，但是可以通过换行整合的方式使之缩小很多，比如将内容写成下面这种

```dockerfile
    RUN dnf install python3 gcc \
        && cat /data/our_hosts > /etc/hosts \
        && cat /log/balabala.log | grep "test" > /data/test_group.log \
        && pip install Django \
        && django-admin startproject testproject
```

将本来要写很多行的整理到一个命令后面，以此来缩减镜像层数

## 镜像的可追溯性

`Dockerfile`比`docker commit`具有更好的可追溯性。所谓的**可追溯性** 就是说可以追溯这个镜像从基础镜像开始到最新的这个版本都做了什么。

如果使用`docker commit`的方式维护镜像，那么可以通过commit时候的备注和shell的历史记录来追溯，但是这两种方式都是不确定的，这样时间长久下去就会导致**祖传镜像** 的出现。所谓祖传镜像就是指那些“我也不知道这个镜像里都改过哪些配置，装过哪些软件和第三方包，反正还能用就凑合用把”的镜像。

如果是使用的`Dockerfile`那么就很容易了，构建这个镜像所经历的所有所有步骤都清清楚楚写在`Dockerfile`里了，可追溯性比`docker commit`高到不知道哪里去了。

# 0X04 精简镜像

精简镜像其实原则很简单，就是**删除不用的东西，减少镜像层数** 。首先正确使用`Dockerfile`就可以大幅度精简镜像了，其次就是在Dockerfile中记得清理一些无用的东西。

  1. 比如你要再容器中编译安装一个C库，那么最后是不是`gcc`就用不到了？用不到了就卸载掉。
  2. 每次`apt install & dnf install & pip install`是不是要有缓存文件？用完了就删掉
  3. 以此类推，将用不到的内容清理掉就行了

比较重要的一点就是选择一个合适的基础镜像，通常来说我们自己使用`ubuntu`搞一个镜像装上`Python`再搞上`Django`(Python的一个web框架)所构建的镜像不会比直接用`Django`的官方基础镜像更好。

# 0X05 需要注意

有几个需要注意的点：

  1. `docker image`是有层数限制的，目前是127层，超出的话再commit是会报错的
  2. `docker build`实际上就是自动的`docker run/xxx/commit/stop/rm`工作流
  3. `docker build .`中的`.`看起来是`Dockerfile`所在位置，实际并不是，而是“上下文环境”。具体的内容比较多，可以自行搜索了解
  4. 指定`Dockerfile`要用`docker buld -f xxx/xxx/xxx/hello .`，其中`xxx/xxx/xxx/hello`被当作`Dockerfile`
