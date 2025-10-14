---
title: "使用 Docker 部署 MySQL 和 Redis"
slug: "docker-mysql-redis"
date: "2019-11-14T12:56:00+0000"
lastmod: "2025-01-16T08:36:02+0000"
draft: false
tags:
  - "Docker"
  - "MySQL"
  - "Redis"
visibility: "public"
---
# 0X00 使用docker部署的优势

在使用docker部署之前，一般都是直接将MySQL和Redis这类服务直接安装在机器上的。以至于好多新手才开始安装使用的时候经常会出问题，出了问题解决不了就重装系统然后再重装软件，而且如果想同时用MySQL5和MySQL8就非常麻烦了。话说回来，在生产环境服务器上其实还是很多直装的服务的，不过其实使用docker部署一套相同的环境是非常有利于自己本地开发的。

我个人看来使用docker部署的优势有这几点比较明显的：

  1. 很容易做到开发环境、测试环境和生产环境的“环境与版本”大统一；
  2. 很容易在开发环境本地同时部署多套不同版本的同一服务；（比如你负责8个项目，这8个项目要用8个不同版本的MySQL）
  3. 能很快部署一套开发环境；（事实证明在一台网络环境好的机器上，能在5分钟内部署一套数据库）
  4. 整理好配置文件后可以很容易备份整套配置；
  5. 安全问题，在物理机上直接装了MySQL，如果部署不够仔细的话有数据库被攻破后危及服务器的风险，而Docker部署的由于容器所在就不会有这种问题；

# 0X01 使用docker部署MySQL

首先把最新的镜像拉下来`docker pull mysql:latest`，然后可以使用`docker run --name your_first_mysql -e MYSQL_ROOT_PASSWORD=your_password -d mysql:latest`这个命令来启动一个MySQL了。

现在来尝试连接一下MySQL吧（其实并不能，现在连端口都没映射出来，不信可以连一下试试）。要想真正连到刚刚的MySQL里的话需要这样操作

```sh
    # 创建MySQL容器
    docker run --name your_first_mysql -e MYSQL_ROOT_PASSWORD=your_password -d mysql:latest

    # 进入到MySQL容器里
    docker exec -it your_first_mysql bash

    # 在容器里
    root@b42296a45a92:/# mysql -p
    Enter password:
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 9
    Server version: 8.0.18 MySQL Community Server - GPL

    Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | sys                |
    +--------------------+
    4 rows in set (0.00 sec)
```

可以发现这次终于连上了，不过这样直接启动还是有诸多问题存在的：容器被删除后数据也就没了、其他程序很难连接到这个MySQL上、只设置了密码还有好多配置没动呢。。。先不管这些个问题，只需要知道现在这个服务已经成功启动了就好，具体的配置会在`docker-compose编排`中介绍。

# 0X02 使用docker部署Redis

首先把最新的镜像拉下来`docker pull redis:latest`，然后就可以用`docker run --name your_first_redis redis -d`这个命令来启动一个Redis了。

现在来试试连接到Redis吧（不知道你发现了没有，我完全就是在重复上面的过程。就算刚才没发现，现在也该发现了。既然你都发现了那我也就不重复了，直接来看docker-compose的编排吧）

# 0X03 使用docker-compose编排

作为开发人员而非运维，`docker-compose`的使用次数是非常高的。通常来说使用Docker都是在Unix like环境中（也就是指的Linux、MacOS或者BSD），那我们安装和使用它就很方便了：`pip install docker-compose --user`就可以了。显然这又是一个Python编写的工具，装好就可以开始编写`docker-compose.yml`文件了下面看一下我完整的这个实例文件

```yaml
    version: "3"
    services:
      mysql:    # 服务的名字
        image: mysql:latest # 使用最新的MySQL镜像
        container_name: dev_mysql_latest  # 指定一个容器名
        restart: always # 自动重启（开机后也自动启动）
        network_mode: bridge  # 指定网络为桥接
        volumes:  # 挂载的目录
          - ./data/mysql:/var/lib/mysql # 将容器中的/var/lib/mysql挂载到当前目录下的data/mysql（MySQL的数据文件，防止容器删除后数据丢失）
          - ./config/mysql:/etc/mysql/conf.d  # 挂载好配置文件的目录（没有特殊配置的时候可以不用写这行）
        ports:
          - "3306:3306" # 将容器的3306端口映射到本地的3306
        environment:
          - MYSQL_ROOT_PASSWORD=test_passwd # MySQL的root密码「必填」

          # 以下皆为选填
          - MYSQL_DATABASE=test_db  # 如指定，则在容器生成时创建该数据库
          - MYSQL_USER=test_user  # 新用户名
          - MYSQL_PASSWORD=test_passwd # 新用户的密码
          - MYSQL_ALLOW_EMPTY_PASSWORD=no # 不允许使用空密码
          - MYSQL_RANDOM_ROOT_PASSWORD=no # 不适用随机密码（为yes时会随机生成一个密码并输出到stdout上，通常是你看到的窗口）
          - MYSQL_ONETIME_PASSWORD=onetime_passwd # 一次性密码（使用时，第一次登录会强制要求修改密码）

      redis:
        image: redis:latest
        container_name: dev_redis_latest
        network_mode: bridge
        restart: always
        ports:
          - "6379:6379"
        command: redis-server --requirepass "mypassword"  # 指定容器运行的命令，命令处设置密码
```

接下来在这个`docker-compose.yml`所在的目录下执行`docker-compose up -d`，这时候两个服务就都在后台悄悄启动好并可以连接使用了。如果不想同时开启所有的服务可以在后面接服务名，例如：`docker-compose up -d redis`。

# 0X04 其他关于docker与docker-compose

  1. `docker exec -it dev_mysql_latest bash`可以接入到`dev_mysql_latest`容器的bash中进行简单的操作；
  2. `docker-compose up -d`中的`-d`是在后台的意思，可以不加这个参数从而把输出都打在终端上方便调试；
  3. `docker-compose stop`和`docker-compose rm`分别是停止容器和销毁容器，容器必须先停止再销毁；
  4. 如果想直接销毁容器可以使用`docker-compose down`，从输出可以看到就是先执行了`stop`再执行了`rm`。
