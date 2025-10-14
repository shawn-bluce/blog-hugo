---
title: "Docker 容器中的文件持久化"
slug: "docker-file-persistence"
date: "2022-05-12T12:39:00+0000"
lastmod: "2025-01-16T08:35:10+0000"
draft: false
tags:
  - "Docker"
  - "Volume"
visibility: "public"
---
# 0X00 最常见的持久化方式：挂载出来

相信各位学习使用 Docker 的时候都会出现过好不容易用 Docker 启动了个数据库容器，然后发现只要容器消失之后数据也就一起消失了的情况。然后通常来说会使用这么一个方法来解决问题：将宿主机的某个目录挂载到容器里，这样一来那个数据库容器就可以将数据内容和配置持久化地存储下来了。一般会使用这么一个方法将某个目录挂载到容器内部 `docker run -dit --name new_container --mount type=bind,source=/Users/shawn/Downloads/test/test_dir,target=/test_dir alpine` 。

下面图中首先创建了一个名为 this_container 的容器，并将其 `/test_dir` 目录和宿主机的 `/Users/shawn/Downloads/test/test_dir` 目录绑定到一起了。然后在容器里向 `/test_dir` 写入文件之后发现在宿主机的对应位置也是可以看到的，并且文件不会因为容器的生命周期结束而被删除（如果数据在容器内部的话当容器被删除后自然也就不在了）。

![将目录挂载到容器内部](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/docker-mount.png)

这种方式其实更适合宿主机和容器需要共享一些文件的时候使用，例如我本地的开发环境在容器里但是代码又在宿主机上的情况就很适合这种方案。当然也可以将数据文件、配置文件等通过这种方式挂载出来。一般使用 Docker 部署各类服务的时候，都会将配置文件通过这种方式共享出来，方便我们在宿主机上修改配置然后在容器内部生效。

# 0X01 另一种持久化的方式：卷

说来惭愧，我一直以为只有上面一种数据持久化的方式，直到昨天看书的时候才知道 Docker 还有一种叫做 Volume 的存在。如果说上面说的将数据挂载出来到宿主机的方式有点类似与宿主机与虚拟机共享目录的话，那通过 Volume 持久化文件就类似于给虚拟机挂载一个虚拟磁盘了。

我们首先可以通过 `docker volume ls` 的方式检查一下当前环境是不是存在已经创建好了的 Volume，通常来说如果你的 Docker 环境已经用过一段时间并且创建过各种不同的容器的话那很有可能已经有存在的 Volume 了。接下来我们通过 `docker run -dit --name volatiner --mount source=bizvol,target=/vol alpine` 来启动一个容器，后面 mount 的参数就是将名为 bizvol 的 Volume 挂载到容器的 /vol 目录上。这里值得注意的一点是即使你没有手动创建这个名为 bizvol 的 Volume 也是可以正常启动这个容器的，Docker 会为我们自动创建这个 bizvol Volume 的。接下来再使用 `docker volume ls` 就可以看到这个被创建好了的 Volume 了，我们尝试着向其中写入一点点数据，然后删除这个容器，可以发现这个 Volume 还是健在的，此时我们再启动一个新的容器使用这个 Volume 是没问题的，甚至启动多个容器同时挂载都是可以的，这些容器里都能正常读写该 Volume 的数据。

![使用 Volume](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/docker-volume.png)

关于 Volume 有如下几个命令，这里简单介绍一下

```sh
    docker volume create    # 创建 Volume
    docekr volume ls        # 查看当前存在的 Volume
    docker volume inspect   # 检查 Volume 的详细信息
    docker volume prune     # 删除未被容器或服务使用的全部 Volume
    docker volume rm        # 删除指定 Volume
```