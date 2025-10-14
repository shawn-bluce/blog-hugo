---
title: "Dockerfile 中的 COPY 与 ADD 指令"
slug: "dockerfile-copy-and-add"
date: "2019-09-25T14:08:00+0000"
lastmod: "2025-01-16T08:36:48+0000"
draft: false
tags:
  - "Docker"
visibility: "public"
---
# 0X00 就算只有一节我也要写标题

众所周知Dockerfile是构建Docker镜像的优良方式，而使用Dockerfile构建镜像最重要的就是为数不多的几个命令，而本次的主题`COPY`和`ADD`就是其中两个。我们知道这两个命令都是将文件搞到Docker镜像里用的，那究竟有没有区别，有什么区别呢？

首先我们看一下当前这个目录：

```sh
    λ  ~/Workstadion/learn_docker  ls
    Dockerfile  excited.tar
```

我们有着么一个Dockerfile，可以看到是基于`fedora`的一个镜像，并且将目录下的`excited.tar`放进了创建好的`shawn`目录中

```dockerfile
    FROM fedora
    RUN mkdir shawn
    WORKDIR shawn
    COPY excited.tar .
```

我们可以看到工作目录下已经存在了一个`excited.tar`了，也就意味着我们成功将这个文件搞进去了。

```sh
    λ  ~/Workstadion/learn_docker  docker build . -t wtf ; docker rm learn_docker; docker run --name learn_docker -it wtf bash
    Sending build context to Docker daemon  23.04kB
    Step 1/4 : FROM fedora
     ---> e9ed59d2baf7
    Step 2/4 : RUN mkdir shawn
     ---> Using cache
     ---> 5014855b8533
    Step 3/4 : WORKDIR shawn
     ---> Using cache
     ---> 82f9d06a4d3c
    Step 4/4 : COPY excited.tar .
     ---> 1bc293d81b33
    Successfully built 1bc293d81b33
    Successfully tagged wtf:latest
    learn_docker
    [root@f6a4f34ed504 shawn]# ls -l
    total 20
    -rw-r--r-- 1 root root 20480 Sep 25 14:15 excited.tar
```

如果同样的操作用`ADD`呢？看上去是类似的操作实际上并不是

```dockerfile
    FROM fedora
    RUN mkdir shawn
    WORKDIR shawn
    ADD excited.tar .
```

我们进到容器里可以看到打包文件被拆解了（压缩文件也会被解压）

```sh
    λ  ~/Workstadion/learn_docker  docker build . -t wtf ; docker rm learn_docker; docker run --name learn_docker -it wtf bash
    Sending build context to Docker daemon  23.04kB
    Step 1/4 : FROM fedora
     ---> e9ed59d2baf7
    Step 2/4 : RUN mkdir shawn
     ---> Using cache
     ---> 5014855b8533
    Step 3/4 : WORKDIR shawn
     ---> Using cache
     ---> 82f9d06a4d3c
    Step 4/4 : ADD excited.tar .
     ---> cc1d067d0dbe
    Successfully built cc1d067d0dbe
    Successfully tagged wtf:latest
    learn_docker
    [root@89276af7b0a2 shawn]# ls -l
    total 4
    drwxr-xr-x 2 1000 1000 4096 Sep 25 14:15 excited
    [root@89276af7b0a2 shawn]# cd excited/
    [root@89276af7b0a2 excited]# ls
    file_0  file_10  file_12  file_14  file_16  file_18  file_2  file_4  file_6  file_8
    file_1  file_11  file_13  file_15  file_17  file_19  file_3  file_5  file_7  file_9
```

其实不止这样，`ADD` 命令还能下载文件：`ADD https://too.young/too/simple.pdf /hello`就能将文件下载下来并且命名为`hello`；如果是`ADD https://too.young/too/simple.pdf /hello/`（只是最后多了斜杠），docker会认为你想将文件下载到`/hello`的目录下，如果没有他会自己创建；还有我们不是说`ADD`能下载文件还能解压文件吗，但是这两个又不能同时生效，意味着如果你想用`ADD https://too.young/too/simple.zip /sometimes/naive/`将下载好的`simple.zip`文件解压到`/sometimes/naive/`目录里是不行的，最后结果只是将压缩包下载到了那个目录而已。

总结来说的话就是 **`COPY`简简单单复制，`ADD`灵活多变**。但是其实我们最好就直接用`COPY`命令，真的需要解压什么的操作多写一个管道符也没有多麻烦。而且其他人看管道符后面接命令参数要比分析`ADD`命令来的舒服得多。

算下来`ADD`能做的没有什么是`COPY`加管道做不了的。而且又加上`ADD`的结果变换多端，不管是自己写还是其他人看都很麻烦，所以说大家能不用`ADD`就不用了吧。
