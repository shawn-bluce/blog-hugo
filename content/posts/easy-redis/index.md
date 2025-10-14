---
title: "Redis入门使用：分库、认证与持久化"
slug: "easy-redis"
date: "2019-09-19T13:59:00+0000"
lastmod: "2025-01-16T08:47:48+0000"
draft: false
tags:
  - "Redis"
visibility: "public"
---
# 0X00 Redis的分库

使用过MySQL或者类似的数据库都应该知道，一个数据库内部是可以分成多个库的。比如MySQL从上到下是`MySQL service -> database -> table -> field`，但是一开始使用redis的时候好像是没有database这一层的呢？其实是存在这么一层的，redis默认是存在编号0到15这总共16个库的，每个库除了命名空间不同以外都是相同的。也就是说在编号为0的库里`set name shawn`之后跑到编号为1的库里`get name`是拿不到的。

那这么说来这个分库究竟有什么用呢？其实很少用的到，甚至就连`redis`的[设计者自己都说搞这个是较蠢的操作](<https://groups.google.com/forum/#!msg/redis-db/vS5wX8X4Cjg/8ounBXitG4sJ>)。我们日常用到的唯一一处地方就是在测试服务器上，因为一台机器部署了太多服务，而且每个服务又都要用redis所以就每个服务分开使用0~15这些数据库。

因为开一个完整redis实例的资源消耗本身就很小，所以分库这个操作就更显的不太用的到了，毕竟我们完全可以在一台机器上开多个redis实例从而实现相同的效果，而且又方便管理。

不过也还是简单介绍一下好了：`/etc/redis.conf`文件中的`databases`参数是设置redis实例具有多少个数据库的，如果真的需要的话可以修改这个参数然后重启redis从而生效。

那么怎么切换这些库呢？我们使用`redis-cli`连上数据库之后可以看到类似这么一个命令提示：

```
    127.0.0.1:6379>
```

这就意味着我们在操作0库，使用`select`指令可以切换库，切换后在命令行上会提示出来

```
    127.0.0.1:6379> select 2
    OK
    127.0.0.1:6379[2]> select 6
    OK
    127.0.0.1:6379[6]> select 15
    OK
    127.0.0.1:6379[15]> select 16
    (error) ERR DB index is out of range
    127.0.0.1:6379[15]>
```

这里可以看到在默认情况下切到一个并不存在的库会报错。（真是一本正经的废话）

# 0X01 Redis的认证

redis的认证要比MySQL要简单的多，在MySQL中要配置用户名、密码甚至还要校验网段，不过redis中就只有一个简简单单的密码校验。而且是那种不需要用户名，仅有密码的校验。还有一点不同的就是，redis不支持设置多个密码。

在默认情况下redis刚刚装完是不需要认证直接使用的，设置密码有下面两种常用方式：

## 修改配置文件

修改`/etc/redis.conf`配置文件，里面有一个`requirepass`参数，默认是加了注释的也就意味着并没有密码。如果要为其加上密码就将那一行改为`requirepass this_is_password`，然后使用类似`systemctl restart redis`的命令来重启一下你的redis服务，密码就生效了。

## 执行redis命令

另一种方法就是使用redis命令`config set`的方式来设置密码，这种方式好在不需要重启redis服务，但是默认情况下一旦重启密码也就跟着失效了。

```
    127.0.0.1:6379> config set requirepass this_is_password
```

## 认证登录

那密码搞上了，怎么登录呢？登录也有两种方式，一种是直接把密码写在连接的命令里，另一种是进入到redis-cli的交互界面再输入密码

```sh
    λ  blog master ✗ redis-cli
    127.0.0.1:6379> keys *
    (error) NOAUTH Authentication required.
    127.0.0.1:6379> exit

    # 直接将密码写在连接的命令行里
    λ  blog master ✗ redis-cli -a this_is_password
    Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
    127.0.0.1:6379> keys *
    1) "a"
    127.0.0.1:6379> exit

    # 进入到redis-cli交互界面后再验证
    λ  blog master ✗ redis-cli
    127.0.0.1:6379> auth this_is_password
    OK
    127.0.0.1:6379> keys *
    1) "a"
```

> 没有验证也能进入到redis-cli的交互界面，只是没有权限而已

> 其中`keys *`可以看到当前库里的所有key

# 0X02 Redis持久化

本来还像要自己写一些关于持久化的初级知识，结果发现[这篇文章](<https://segmentfault.com/a/1190000002906345>)已经写的超级棒了，索性补充一小部分他这里没有提到的吧。

  1. 默认情况下redis装好就启用了RDB；
  2. 默认情况下重启redis的时候会写入一次RDB；

**使用docker部署redis需要注意的点** ：如果使用docker部署redis，并且需要对数据进行持久化的话一定记得将redis数据文件挂载出来，否则会导致数据丢失。因为持久化的文件存在与docker容器内部，只是重启容器还好问题不大，但是如果`docker stop/rm/run`一波下来就会导致容器内部的数据被删的一干二净了（准确地说容器都不是之前的那个了，数据当然也就不在了）。
