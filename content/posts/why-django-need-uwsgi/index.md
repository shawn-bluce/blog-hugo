---
title: "为什么 Django 需要uWSGI"
slug: "why-django-need-uwsgi"
date: "2020-06-30T13:35:00+0000"
lastmod: "2025-01-17T02:15:26+0000"
draft: false
tags:
  - "Django"
  - "uWSGI"
visibility: "public"
---
# 0X00 运行一个 Django 程序

运行一个 Django 程序可太简单了，从创建项目到运行起来总共也不超过 5 行代码。项目运行起来了就可以打开我们的 vim 或者 IDE 之类的一顿 coding 了。作为最最最开始写 Django 的同学来说到这里也就了解的差不多了，因为大家都是自己写好代码本地测试一下就提 Pul Request 到上游仓库了，然后什么单元测试、数据库迁移、测试环境版本发布甚至可能包含 docker 镜像更新就全都交给 CI 来做了。自己就这么开开心心的写了一段时间的代码，一切都在朝着好的方向发展。突然有一天部门主管或者老大告诉你有一个新项目要你来开个头，先搭好脚手架然后发布上去，后面再来人一起做功能迭代。

![运行一个 Django 程序](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/20200630214416.png)

然后你开开心心地`django-admin startproject xxxx`、开开心心地`django-admin startapp xxxxx`，一顿 coding 之后懵逼了，没有部署过测试环境，没有部署过生产环境，只知道 CI 给做了，却完全不知道做了什么。然后你跑去看 CI 脚本，去问其他同事同学，得到了一堆 Nginx 和 uWSGI 之类的答复。你也照着做了，但是完全不知道为什么，因为你觉得`python manage.py runserver`明明就可以启动项目了，为什么还需要搞什么 Nginx 和 uWSGI 呢？

# 0X01 部署到生产环境

按我现在手上的项目来说，部署阶段用到了：docker、uWSGI、supervisor、Nginx、fabric、gitlab-ci 这 5 种技术。其中 docker 是为了方便大家的生产环境、测试环境和开发环境高度一致的；uWSGI 和 Nginx 后面单独说；supervisor 是用来保活的，有时候进程挂了需要有人来重启它；fabric 是用来将常用的命令组整合的；gitlab-ci 是用来方便运行单元测试、测试环境发布和生产环境发布的。

![简单部署 Django 程序](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/20200630221308.png)

下面就重点来解答一下这个新手群体中很容易出现的疑问：

> 为什么明明`python manage.py runserver`就能运行的 Django 项目非要用 Nginx 和 uWSGI 运行呢？

# 0X02 为什么需要 uWSGI

具体为什么要用 uWSGI 其实偶遇好多个原因，下面会逐个提到。

首先最常见的一种说法是：“`python manage.py runserver`是单线程的，一个请求不结束其他的请求就得阻塞，所以性能差”，这种说法彻头彻尾是错误的。而且验证起来也及其容易，随便找一个你的 Django 项目然后找个 api 用 ipdb 打个断点，通过 runserver 把它跑起来。然后先调用打了断点的 api，发现它卡在断点处了，然后再调另一个 api，实际上是可以得到 response 的，所以并不是所谓单线程导致的。与其说是因为单线程导致性能低下倒不如说是因为“单进程”，众所周知 Python 中有一个 GIL 全局解释锁，这东西的存在导致真正意义上的单进程多线程 Python 程序并不得行，所以 uWSGI 可以通过启动多个进程的方式来规避这个问题，也就是下面这个配置文件中`processes = 8`的部分。

```conf
    [uwsgi]
    chdir     = /code
    pp        = /code
    module    = ratel.wsgi
    master    = true
    processes = 8
    vacuum    = true
    http      = 0.0.0.0:7701
    stats = /tmp/xxxxx_stats.socket

    env = LANG=en_US.utf8
    env = DJANGO_SETTINGS_MODULE=xxxxx.settings

    harakiri  = 60
    http-timeout   = 60
    socket-timeout = 60

    max-requests = 8192
    listen       = 8192
    no-orphans
```

> 这里说到多进程，有个题外话可以说一下：“你觉得 Django 中使用全局变量有什么需要注意的吗？”。通常情况来说编程中使用全局变量是一个很常见的问题，但是在多进程的情况下就需要多多注意了。比如按上面的配置，一个项目用了 8 个进程，其中一个进程接到了一个 request 并且把`CURRENT_COUNT`从 80 改到了 100，但是其他进程在访问自己进程里的`CURRENT_COUNT`的时候还是 80，这就是因为多个进程中全局变量不够“全局”导致的问题。那么怎么解呢？其实也简单。我们还是用`CURRENT_COUNT`，但是不用它做变量名了，而是 MySQL 的配置表中的一条数据，或者 Redis 中的 key 就能解决这个问题了。

性能问题当然是个大问题，不用 runserver 而是改用 uWSGI 的一大原因就是性能问题，我们用 `ab` 来测试一下 runserver 和 uWSGI 两种方式启动项目的性能差距，`ab -n 1000 -c 100 -p data -T application/json http://127.0.0.1:7070/remit_verify/fetch_order/`意思为总共 1000 次请求，并发 100，访问后面的登陆接口。首先看一下使用 runserver 方式运行的 Django 成绩：

```
    Concurrency Level:      100
    Time taken for tests:   53.702 seconds
    Complete requests:      1000
    Failed requests:        0
    Non-2xx responses:      1000
    Total transferred:      269000 bytes
    Total body sent:        167000
    HTML transferred:       63000 bytes
    Requests per second:    18.62 [#/sec] (mean)
    Time per request:       5370.198 [ms] (mean)
    Time per request:       53.702 [ms] (mean, across all concurrent requests)
    Transfer rate:          4.89 [Kbytes/sec] received
                            3.04 kb/s sent
                            7.93 kb/s total

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    0   0.6      0       3
    Processing:    18 1574 6329.8    113   53693
    Waiting:        6 1567 6330.6    107   53691
    Total:         18 1575 6330.2    114   53695

    Percentage of the requests served within a certain time (ms)
      50%    114
      66%    133
      75%    147
      80%    177
      90%   1543
      95%   7727
      98%  27565
      99%  53662
     100%  53695 (longest request)
```

然后再来看一下使用 uWSGI 的成绩：

```
    Concurrency Level:      100
    Time taken for tests:   2.223 seconds
    Complete requests:      1000
    Failed requests:        0
    Non-2xx responses:      1000
    Total transferred:      231000 bytes
    Total body sent:        167000
    HTML transferred:       63000 bytes
    Requests per second:    449.93 [#/sec] (mean)
    Time per request:       222.258 [ms] (mean)
    Time per request:       2.223 [ms] (mean, across all concurrent requests)
    Transfer rate:          101.50 [Kbytes/sec] received
                            73.38 kb/s sent
                            174.87 kb/s total

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    0   0.8      0       4
    Processing:    10  213  30.9    222     298
    Waiting:        6  213  31.0    222     297
    Total:         10  214  30.5    222     300

    Percentage of the requests served within a certain time (ms)
      50%    222
      66%    225
      75%    226
      80%    227
      90%    229
      95%    231
      98%    250
      99%    274
     100%    300 (longest request)
```

其中使用 runserver 方式运行压力测试总共耗时 53.7 秒，uWSGI 耗时 2.2 秒。使用 uWSGI 后吞吐率是 24 倍，用户平均等待时间是 1/24，服务器处理时长也是 1/24 的样子。综合来说在我自己配置情况下得到了答曰 24 倍的性能提升，我这儿还是只有双核双进程的情况，如果我给 docker 配置允许使用所有 12 个逻辑处理器，情况会更显著。

# 0X03 为什么需要 Nginx

至于为什么要用 Nginx 就更简单了，有这么几个原因：静态资源、HTTPS、端口、多服务部署。。。。。。

首先是静态资源，web 服务无可避免的会用到静态资源，就算前后端分离得再彻底也不免有一些从后端服务上获取一些静态资源，尤其是用户上传下载的文件。Python 的性能咱们也抖动，如果不需要对文件进行权限限制的话，的确没必要让这些请求再走一遍 Django 了。

然后是 HTTPS，这个东西如果不在 Nginx 或者 Apache 上配置的话，就会很麻烦，没这个必要。

端口问题，有时候我们并不打算让服务暴露的是 80 或者 443，尤其是当服务在容器中的时候，这时候就需要用 Nginx 来做代理转发了。

多服务部署就更是需要 Nginx 了，如果你只有一台服务器但是需要部署四五个 web 服务怎么办？总不能说每个服务暴露不同的端口然后让用户根据不同的端口来决定访问哪个服务吧。所以还是需要用 Nginx 来做，根据不同的域名来控制转发到本地的哪个端口上去。
