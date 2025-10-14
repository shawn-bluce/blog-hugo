---
title: "部署 Kubernetes 集群时遇到的一些问题"
slug: "k8s-deploy-tips"
date: "2022-12-08T12:30:00+0000"
lastmod: "2025-01-16T09:10:49+0000"
draft: false
tags:
  - "Kubernetes"
  - "Container"
visibility: "public"
---
# 0X00 🛡️叠buff

  * 我是个 Kubernetes 纯新手，并不懂很多原理和概念，可能会有误导；
  * 我只是将自己遇到过的问题列出来，如果你想找「权威」请看官方文档；
  * 每个人的基础环境不同，Kubernetes 版本也不同，参考应该学会变通；
  * 此处记录的仅为个人部署过程中遇到的问题记录，并非「教程」或「指南」；

# 0X01 正文

## 注意交换分区

部署 Kubernetes 的节点是不允许使用交换分区的，临时禁用可以 `swapoff -a`。然后在 `/etc/fstab` 中将交换分区的自动挂载给注释掉就可以了。

![k8s_swap](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/12/08/k8sswap.png)

## 不使用 docker

Kubernetes 已经不建议使用 docker 作为容器运行时了，可以考虑使用 `containerd` 或者 `CRI-O`。注意这里提到的 Dockershim 已经移除并不意味着不能再用 Docker 了，而是说 Kubernetes 并不会原生支持 Docker 了，以后 Docker 的地位和其他运行时的地位相同了。

![k8s-container-runtime](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/12/08/k8scontainerruntime.png)

## Containerd 的默认配置

如果使用 Containerd 作为容器运行时的话，安装好 Containerd 之后要检查一下配置文件 `/etc/containerd/config.toml`。如果没有的话需要手动创建一个默认配置

```sh
    mkdir /etc/containerd
    containerd config default > /etc/containerd/config.toml
```

## kubelet 疯狂重启

如果你还没有搞定 `kubeadm init` 的话，kubelet 疯狂重启是正常行为，它在等待 kubeadm。

## 需要 AppArmor

我之前手贱把 AppArmor 给卸掉了，自以为是得认为它会影响到 Kubernetes 的运行。其实它确实会影响，因为 Kubernetes 的运行时需要它的，所以后来看日志在报错就又装回来了。

## 正确配置 cni 网络插件

如果你是使用 Containerd 的话，可以参考 <https://github.com/containerd/containerd/blob/main/script/setup/install-cni>

## 容器运行时的冲突

如果要用 Containerd 的话就记得先把 docker 卸载干净，否则可能会导致一些冲突和不兼容的状况。

## 正确加载内核模块和内核参数

我当时使用的是 Ubuntu 22.04，默认没加载 `br_netfilter` 模块，所以需要使用 `modprobe br_netfilter` 临时加载，并且使用 `echo br_netfilter >> /etc/modules` 持久化加载，保证其重启后也是被加载的。可以使用 `lsmod | br_netfilter` 的方式检查是否已经加载了该模块。

然后就是内核参数，首先可以通过 `sysctl -a | grep xxx` 的方式检查当前参数是否设置正确了。如果没设置正确的话需要在 `/etc/sysctl.conf` 中进行修改。改好之后 `sysctl --system` 可以使改动生效。

```conf
    net.ipv4.ip_forward=1
    net.bridge.bridge-nf-call-iptables=1
    net.bridge.bridge-nf-call-ip6tables=1
```

## 使用 crictl 命令

如果你想使用 `crictl` 命令查看容器/镜像，则需要修改其配置文件让它能够连接到你的容器运行时，配置文件在 `/etc/crictl.yaml`。如果你用的是 Containerd 的话配置文件可以写成

```yaml
    runtime-endpoint: unix:///run/containerd/containerd.sock
    image-endpoint: unix:///run/containerd/containerd.sock
    timeout: 10
    debug: true
```

## 关于认证文件

当 `kubeadm init` 成功之后，记得关注一下它打出来的内容，里面包含了你该如何让其他节点加入，还包含了如何从其他机器上管理 Kubernetes 集群。

![kubeadm_init](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/12/08/kubeadminit.png)

## get nodes

当使用 `kubectl get nodes` 命令查看到的所有节点都处于 Ready 并且过一两分钟也没有异常状态时，也就意味着你的 Kubernetes 集群已经部署完成了🎉

## 关于 dashboard

刚刚部署了一个 kubenetes-dashboard 结果 401/403/40x 了？可以尝试使用端口转发的方案：<https://github.com/kubernetes/dashboard/issues/5542#issuecomment-706395744>

dashboard 打开了，但是没有 Token 登陆不了？那就创建一个 token 喽 <https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md>
