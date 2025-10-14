---
title: "使用 uwsgi 和 supervisor 部署 Django 程序"
slug: "uwsgi-supervisor-dployer-django-app"
date: "2017-12-23T10:05:00+0000"
lastmod: "2025-01-16T08:31:28+0000"
draft: false
tags:
  - "Django"
  - "Python"
  - "uWSGI"
  - "Supervisor"
visibility: "public"
---
# 0X00 使用uwsgi启动Django

首先安装uwsgi，`pip install uwsgi`就可以装好．然后找到Django生成的`wsgi.py`文件，这文件通常实在与项目名同名的app目录下的，比如我的项目名为`django_test`那么这个文件应该就在`django_test/wsgi.py`．然后执行`uwsgi --http 0.0.0.0:8080 --wsgi-file django_test/wssgi.py`就可以用uwsgi启动你的Django项目了.
Django自带的`python manage.py runserver`用于调试还是可以的，不过如果用于生产环境的不论是安全性还是性能都不足以满足生产环境的需要.

# 0X01 使用supervisor维持服务在线

我们知道由于Linux系统的进程管理机制，导致在bash里启动的程序如果退出bash就会自动被kill掉．所以我们需要一个能让进程一直活下去的方法，其中`nohup`是我们常用的方法，不过这种方法也只是能让进程活在后台罢了，平时跑个脚本或是其他小程序还可以，如果是部署一个服务的话就不合适了．这里我们选用`supervisor`来维持服务．

这个工具也是Python写的，所以可以使用pip安装`pip install supervisor`．然后我们可以在任意位置创建一个`supervisor`的配置文件`django_test_supervisor.conf`，启动supervissor的时候会指定这个文件，所以这个配置文件不像是其他配置那样全局使用．配置文件简单中的命令和日志需要自行修改，其中命令要修改成自己使用uwsgi启动Django的命令，日志位置要存在，不要没有那个目录．

```conf
    [unix_http_server]
    file=/tmp/supervisor.sock   ; (the path to the socket file)

    [inet_http_server]         ; inet (TCP) server disabled by default
    port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for all iface)

    [rpcinterface:supervisor]
    supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

    [supervisorctl]
    serverurl=unix:///tmp/supervisor.sock ; use a unix:// URL  for a unix socket

    [supervisord]
    nodaemon=false  # 值为false时supervisor在后台运行
    logfile=/data/log/supervisord.log   # supervisor　的日志
    pidfile=/data/supervisord.pid       # supervisor　的日志


    [program:django_test]
    command=uwsgi --http 0.0.0.0:8080 --chdir /home/shawn/django_test --wsgi-file django_test/wsgi.py  ＃　要执行的命令，其中chdir指定的是项目的目录
    stopsignal=HUP
    stopasgroup=true
    killasgroup=true
    autorestart=true
    stdout_logfile=/data/log/uwsgi.log  # uwsgi　的日志
    stderr_logfile=/data/log/uwsgi.log  # uwsgi　的日志
    stdout_logfile_maxbytes = 20MB
    stderr_logfile_maxbytes = 20MB

    [group:django_test]
    programs=django_test
```

写好配置文件之后启动supervisor，`supervisord -c django_test_supervisor.conf`就可以把服务启动起来了．

启动起来之后可以进入控制台去管理任务．`supervisorctl -c django_test_supervisor.conf`可以进入到supervisor的命令行，并且可以看到管理的进程．可以通过`start django_test/stop django_test/restart django_test`等命令去操作进程．

# 0X02 后记

现在服务已经可以直接上线了，但是通常情况下我们会让Nginx代理一下到uwsgi，这样静态文件什么的就不会再走一次Python程序了．
