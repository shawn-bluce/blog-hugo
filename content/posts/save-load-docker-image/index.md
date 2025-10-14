---
title: "Docker 中备份与恢复镜像"
slug: "save-load-docker-image"
date: "2018-02-15T06:56:00+0000"
lastmod: "2025-01-16T08:34:12+0000"
draft: false
tags:
  - "Docker"
visibility: "public"
---
# 0X00 遇到了一个问题

前段时间自己的电脑重装了系统，然后公司内网的Docker hub出了点问题，没办法继续开发。后来经过一波Google找到了一个可以备份与恢复Image的方法，使用`docker save / docker load`命令。

# 0X01 备份与导入镜像

首先查看自己的镜像，然后找一个准备备份的镜像，找到他的`IMAGE ID`，假设选中的是`ubuntu`的镜像，那就使用`dokcer save 0458a4468cbc --output ubuntu.tar`就可以把ubuntu.tar文件拷贝到其他电脑上。

```
    REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
    nextcloud                      latest              9d7d01184cbf        9 days ago          593MB
    rabbitmq                       latest              72cee1616e73        2 weeks ago         127MB
    ubuntu                         latest              0458a4468cbc        2 weeks ago         112MB
    redis                          latest              861cc310cd91        3 weeks ago         107MB
    mysql                          5.7.17              9546ca122d3a        10 months ago       407MB
    mongo                          3.4.2               5bc602c0b7fe        10 months ago       360MB
```

到另外一台电脑上执行`docker load --input ubuntu.tar`就可以把这个镜像导入进去。
