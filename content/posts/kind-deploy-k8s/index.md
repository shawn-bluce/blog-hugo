---
title: "使用 kind 飞快的创建一个 Kubernetes 集群"
slug: "kind-deploy-k8s"
date: "2022-11-28T13:10:00+0000"
lastmod: "2025-03-04T08:51:03+0000"
draft: false
tags:
  - "Kubernetes"
  - "Container"
  - "Docker"
visibility: "public"
---
# 0X00 为嘛用 kind

作为一个纯新手想要学习 Linux、MySQL、Python... 的第一步往往都是先装一个来看看，当然 Kubernetes 也不例外。装 Linux 也许跟着教程在虚拟机里一会儿就装好了，尤其是现在很多发行版本都有图形化安装界面了，但是 Kubernetes 就不一样了，如果你去部署一套 K8S 集群的话极有可能会遇到一系列问题，包括但不限于：

  * 交换分区没关，导致服务异常
  * 搞不清楚 CRI-O/docker/containerd 之间的关系
  * 刚装好的 `kubelet` 服务疯狂重启
  * cni 网络插件搞不明白，导致 `kubeadm init` 一直不成功
  * 缺少内核模块导致的集群初始化异常
  * 缺少内核参数导致的集群初始化异常
  * 想用 `crictl` 却怎么都看不到 `container` 状态

这些在熟手眼里可能根本不是问题，但是对于第一次接触的人来说还是挺麻烦的。所以我们需要一种简单的方式来快速部署一个 Kubernetes 来看一看试一试，而不是上来就先部署一套三四个节点的真正意义上的集群。

Kubernetes 官方推荐了几种单节点/单机部署的方式，kind 就是其中一种。它是基于 docker 的部署方式，如果你还不熟悉 docker 的话建议先把 docker 学一学再开始 Kubernetes 的学习。如果你已经了解 docker 并能简单的使用它了，那接下来就开始安装 Kubernetes 吧~

tips: 建议在虚拟机中安装，分配 2C4G 然后随便装一个比较新比较常用的 Linux 发行版本就可以

# 0X01 安装

既然是基于 docker 的那自然要先安装 docker 了，常见三种安装方式：

  1. 直接 `apt install docker` ❌ 不推荐
  2. 按照官网操作：新增 mirror、更新 cache、安装 docker ✅ 推荐
  3. 使用官方脚本 <https://get.docker.com/> ✅ 推荐

第二步是安装 kubectl ，它是用来管理 Kubernetes 集群的，类似于 docker 体系中的 docker client 吧。原则上是可以不装 kubectl 的，没有它集群也能正常工作，但没法操作集群的话那还怎么学呢。因为 kubectl 只是一个单纯的二进制文件，所以「安装」就仅仅意味着：下载、挪到 $PATH 中、赋予执行权限。

```sh
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

    chmod +x kubectl

    mv kubectl /usr/local/bin/.
```

装好 kubectl 后就该装 kind 了，kind 方便就方便在它也是一个二进制文件。所以我们去 [GitHub 的 release 页面](<https://github.com/kubernetes-sigs/kind/releases>)找到自己平台最新的二进制包下载，然后依旧是赋权和移动位置就搞定了。

![kind-download](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/11/28/kinddownload.png)

（可选）最后可以将 `source <(kind completion bash)` 和 `source <(kubectl completion bash)` 追加到 `~/.bashrc` 最后，再手动 `source ~/.bashrc` 一下，就可以为这两个命令提供自动补全了。

到此为止 kind 就安装完成了，接下来可以简单介绍一下它的用法。

# 0X02 使用

首先可以使用 `sudo kind create cluster` 来创建一个单节点的 Kubernetes（后面可以接 `--name` 参数为集群取名，默认叫做 `kind`）。因为是基于 docker 的所以需要从 dockerhub 拉镜像，如果「网络通畅」的话速度会比较快，如果不通畅的话可以用更科学的姿势上网~

![kind-create-cluster](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/11/28/kindcreatecluster.png)

也可以使用配置文件的方式来创建集群，例如下面的配置，使用 `sudo kind create cluster --config CONFIG_FILE.yaml`

```yaml
    kind: Cluster
    apiVersion: kind.sigs.k8s.io/v1alpha3
    kubeadmConfigPatches:
    - |
      apiVersion: kubeadm.k8s.io/v1beta1
      kind: ClusterConfiguration
      metadata:
        name: config
      networking:
        serviceSubnet: 10.0.0.0/16
      imageRepository: registry.aliyuncs.com/google_containers
      nodeRegistration:
        kubeletExtraArgs:
          pod-infra-container-image: registry.aliyuncs.com/google_containers/pause:3.1
    - |
      apiVersion: kubeadm.k8s.io/v1beta1
      kind: InitConfiguration
      metadata:
        name: config
      networking:
        serviceSubnet: 10.0.0.0/16
      imageRepository: registry.aliyuncs.com/google_containers
    nodes:
    - role: control-plane
```

当然如果你不想管这么多配置的话也可以用「极简配置」

```yaml
    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    nodes:
    - role: control-plane
```

还可以创建「多个节点」的集群，因为仍然是在同一台机器上，所以用了引号，并非传统意义上的集群。不过创建多节点集群必须要用配置文件了。

```yaml
    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    nodes:
    - role: control-plane
    - role: worker
    - role: worker
    - role: worker
    - role: worker
```

![kind-multiple-nodes](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/11/28/kindmultiplenodes.png)

其他 tips：

  * 使用 `kind get kubeconfig` 可以获取当前的配置文件，可以给 `kubectl` 用
  * 使用 `kind delete cluster` 可以删除默认集群，也可以接 `--name` 删除指定集群
  * 可以同时创建多个集群，不重名就行，可以用 `kind get clusters` 查看当前存在的集群

使用 kind 还能做到：高可用集群、端口映射、指定 Kubernetes 版本、配置代理等等，不过具体的就要去看[官方的简明文档](<https://kind.sigs.k8s.io/docs/user/quick-start/>)喽
