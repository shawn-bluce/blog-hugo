---
title: "MySQL 慢查询初步"
slug: "mysql-slowquer-simple"
date: "2019-11-03T12:17:00+0000"
lastmod: "2025-01-16T09:45:43+0000"
draft: false
tags:
  - "MySQL"
visibility: "public"
---
# 0X00 IO总是比运算慢

众所周知计算机的IO都要比计算慢很多很多，即时是目前民用的高级SSD：三星970PRO，它的读写速度都要比内存慢上几个数量级，更不要说CPU了。所以软件的IO通常都是瓶颈，很多时候都是CPU等内存，内存等磁盘，磁盘等网络。

那么如何才能提升自己web服务的响应速度呢？通常来说简单的操作有如下两种：换硬盘或者改SQL。

# 0X01 换硬盘

“这难道不是废话吗？”对呀，这就是废话。当瓶颈出现在数据库的查询上了，那么把正在用的机械硬盘换成固态硬盘当然会提升效率，稍微想想就呢能明白的事情。事实上也是这样的，之前我把同样量级的数据从我们的测试环境搞到我本地，测试环境是企业级HDD，而我本地是三星970EVOPlus的SSD，会发现查询同一个内容就快了好多好多。

那其实这个换硬盘并不是好办法，毕竟不能指望全都用上SSD。而且即使用上SSD了，在查询更复杂或者数据量更多的情况下还是会出现瓶颈。那首先想到的方式就是优化SQL了。

# 0X02 慢查询

大家都知道优化SQL，那么优化哪条呢？一个系统里可能有几千条SQL，总不能一个个看吧。而且现在还有很多很多项目用上了ORM，根本不在系统里写原生SQL了。

这时候MySQL的慢查询功能就帮得上忙了。顾名思义“慢查询”就是很慢的查询，我们通过简单的配置能够让MySQL记录下很慢的查询语句，通过整理分析再回去系统中找到产生这些慢查询的位置逐个优化就可以了。

# 0X03 开启

通常来说开启MySQL慢查询日志记录的方式有两种：直接改配置文件和修改全局变量。

改配置文件：

```conf
    [mysqld]
    slow_query_log = ON     # 开始记录慢查询日志
    slow_query_log_file = /usr/local/mysql/data/slow.log     # 慢查询日志的位置
    long_query_time = 1     # 对“慢”的定义，这里是耗时超过1s       蛤？这很暴力吗？
···

修改配置文件的好处在于不论怎么重启数据库服务，这项配置都是存在的；缺点在于想让其生效需要重启一次数据库才行。

改全局变量：

```sql
    mysql> set global slow_query_log='ON';  // 打开慢查询记录

    mysql> set global slow_query_log_file='/home/shawn/slow_query.log';     // 慢查询日志的位置（我乱写的，不建议写在自己的$HOME下）

    mysql> set global long_query_time=1;    // 还是1s
```

修改全局变量的优势和缺点正好与上面相反，能立即生效但是重启会失效。

# 0X04 使用

我们来检查一下这个是不是配置好了：

```
     ~/Workstadion/blog  master ?1  mycli -h 127.0.0.1 -uroot
    Password:
    mysql 5.7.17
    mycli 1.19.0
    Chat: https://gitter.im/dbcli/mycli
    Mail: https://groups.google.com/forum/#!forum/mycli-users
    Home: http://mycli.net
    Thanks to the contributor - Ryan Smith
    mysql root@127.0.0.1:(none)> show variables like 'slow_query%';
    +---------------------+--------------------------------------+
    | Variable_name       | Value                                |
    +---------------------+--------------------------------------+
    | slow_query_log      | ON                                   |
    | slow_query_log_file | /home/shawn/slow.log                 |
    +---------------------+--------------------------------------+
    2 rows in set
    Time: 0.042s
    mysql root@127.0.0.1:(none)> show variables like 'long_query_time';
    +-----------------+-----------+
    | Variable_name   | Value     |
    +-----------------+-----------+
    | long_query_time | 1.000000 |
    +-----------------+-----------+
    1 row in set
    Time: 0.035s
```

现在看起来我们的慢查询日志就搞好了，开始试一试真正意义上的慢查询呢`SELECT SLEEP(5)`。其实这是个没有意义的查询（不太好拿测试数据出来给大家展示），不过没关系了，因为它会很扎实得等5s，满足我们对“慢查询”的认知了。

然后去看看我们的日志呢，可以看到如下输出，就没有问题了。

```
    root@1971e980abad:/var/lib/mysql# tail -f 1971e980abad-slow.log
    mysqld, Version: 5.7.17 (MySQL Community Server (GPL)). started with:
    Tcp port: 3306  Unix socket: /var/run/mysqld/mysqld.sock
    Time                 Id Command    Argument
    # Time: 2019-11-03T12:47:41.827832Z
    # User@Host: root[root] @  [172.17.0.1]  Id:    14
    # Query_time: 12.000459  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 0
    SET timestamp=1572785261;
    select sleep(12);
```

这时候就可以将慢查询日志的时间设置成一个合理的值，然后就可以静待生产环境或者测试环境打一大堆log出来喽。我们可以看到每个超出我们定义的时长的SQL和其耗时，当日志累计起来我们就可以从耗时最长的或者出现最频繁的开始优化SQL了。

当然，有很多第三方的慢查询日志分析工具可以帮助我们，不过我这种初级MySQL用户遇到的也还不太需要。每次的慢查询日志自己搞下来逐行看一看总结一下规律也就能找到问题所在啦。如果后面日志越来越多越来越复杂的时候再考虑用第三方工具吧~

> 这里连接MySQL用的不是mysql的官方命令行工具，而是一个Python编写的叫做`mycli`的工具。这个工具实现了`mysql`命令的几乎所有功能，并且支持语法高亮与自动补全，使用体验非常棒。可以直接用`pip install mycli --user`安装
