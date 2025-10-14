---
title: "使用 Docker 部署 Sentry 服务"
slug: "docker-deploy-sentry"
date: "2018-08-08T14:39:00+0000"
lastmod: "2025-01-16T09:00:43+0000"
draft: false
tags:
  - "Docker"
  - "Sentry"
  - "Python"
visibility: "public"
---
# 0X00 Sentry是什么

Sentry是一个统一收集整理程序异常错误的服务。如果你有一个程序在跑，并且配置了日志，那么可以轻松的找到程序出错的地方；甚至可以在报错后发邮件通知自己以便抓紧处理。但是如果你的团队有10个项目和50个人，并且这50个人并不是每人只负责一个项目，此时此刻该怎么办呢？难道为每个项目都配置很多人，并且在人员变动和项目变动的时候都去再修改吗？这样就未免有点傻了，Sentry就是用来做这个的。

你可以在Sentry上为每个项目创建一个Project用于收集项目的错误，再为每位成员创建用户，由用户去关注项目，就可以实现上述复杂的功能了。

# 0X01 如何部署

为了流程简洁，默认安装了`docker`和`docker-compose`。

```sh
    # 将一个配置好的使用docker-compose部署Sentry给clone下来
    git clone https://github.com/getsentry/onpremise.git

    cd onpremise

    # 创建所需的目录
    mkdir -p data/{sentry,postgres}

    # 生成一个secret-key
    docker-compose run --rm web config generate-secret-key
```

上述命令执行完成后，输出的最后一行类似乱码的东西是我们所需的secret-key，将其复制粘贴之`docker-compose.yml`文件的`SENTRY_SECRET_KEY`后面，形似


    SENTRY_SECRET_KEY: '*********************'
    SENTRY_MEMCACHED_HOST: memcached


然后初始化数据库`docker-compose run --rm web upgrade`，执行过程中会要求创建一个管理员账户，根据流程的提示操作既可。操作完成后执行`docker-compose up -d`就可以将整套Sentry服务启动起来了。此时如果想要访问Sentry的web服务需要访问`ip:9000`，可以按照下面的方法为其配置nginx的反向代理。

# 0X02 Nginx反向代理

向nxing配置的`http`模块内部添加下面的配置，并重新载入/重启Nginx后就可以使用域名访问Sentry了。注意将其中的`xxxx.example.com`改为自己解析了的域名。

```conf
    server {
        listen 80;
        server_name xxxx.example.com;

        location / {
            proxy_pass  http://127.0.0.1:9000;
            proxy_set_header Host $host;
            proxy_set_header  X-Real-IP $remote_addr;
            proxy_set_header  X-Forwarded-For $remote_addr;
            proxy_set_header  X-Forwarded-Host $remote_addr;
        }

        client_max_body_size 10m;
    }
```

到此为止Sentry就算是配置完成了。

# 0X03 集成到自己的程序中

在Sentry的web服务中创建一个新的Project，创建时需要选择自己程序的类型，比如是`Django`还是`Flask`或是其他，创建好后会有提示，按照提示就可以将Sentry集成到自己的程序中了。过程都非常简单，比如在纯Python程序中集成一个Sentry也就四五行的事情，而在Django中也不过10行代码。

# 0X04 配置发送邮件

此时的Sentry还只能接收错误，如果想要使其能够发送邮件，需要修改`docker-compose.yml`文件，找到几个以`SENTRY_XXX`开头的地方，在这里配置好自己需要使用的发件配置，例如

```yaml
        SENTRY_SERVER_EMAIL: sender@example.com
        SENTRY_EMAIL_HOST: smtp.example.com
        SENTRY_EMAIL_PASSWORD: examplepassword
        SENTRY_EMAIL_USER: sender@example.com
        SENTRY_EMAIL_PORT: 587  # 如果使用TLS加密且采用465端口失效时可以尝试将端口改为587试试
        SENTRY_EMAIL_USE_TLS: 'True'
```

然后把生成的容器停止删除并重建一下，就可以了。

```sh
    docker-compose stop
    docker-compose rm
    # 上面两个操作在新版本的docker-compose中可以使用docker-compose down来替代

    docker-compose up -d
```

# 0XFFFF Done!

整体的大方面的配置就是这样了，具体的可以慢慢在web界面摸索一下，就这样啦

\口-口/
