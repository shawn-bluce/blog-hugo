---
title: "在 Django 中使用 RabbitMQ"
slug: "django-rabbitmq"
date: "2019-11-21T14:02:00+0000"
lastmod: "2025-01-16T08:14:33+0000"
draft: false
tags:
  - "Django"
  - "Python"
  - "RabbitMQ"
visibility: "public"
---
# 0X00 使用docker部署RabbitMQ

自从用起docker之后，每次在自己本地开发环境部署新服务就首选用docker了。虽然理论上docker跟裸机部署比起来多多少少有一些缺点，但是跟3分钟部署几乎一些开发环境服务的优势比起来简直都是毛毛雨了。

首先要拉个镜像下来，通常拉镜像都是选择最新的或者特定某个版本，但是RabbitMQ有一点比较奇怪，如果逆向拉带有web管理页面的就不能用`latest`，而应该选择`management`。然后确定好镜像之后再了解一下端口情况，RabbitMQ带有web管理页面的话会用到两个端口：提供MQ服务的`5672`和提供web服务的`15672`。

下面是我的配置文件，把内容保存为`docker-compose.yml`然后`docker-compose up -d`就好了（如果不在yml文件所在目录下执行或者文件名不叫`docker-compose.yml`的话要用`docker-compose -f xxx/xxx/xxx/xxx.yml`指定配置文件的位置。

```yaml
    version: '3'

    services:
      rabbitmq:
        # image: rabbitmq:latest    # 如果要用不带web界面的可以选这个
        image: rabbitmq:management  # 带有web界面的镜像
        container_name: rabbitmq    # 取一个容器名
        ports:  # 开放两个端口，当然没有web界面的话就不用开放15672了
          - "5672:5672"
          - "15672:15672"
        environment:    # 这里设置登录名和密码
          RABBITMQ_DEFAULT_USER: shawn
          RABBITMQ_DEFAULT_PASS: ****************   # 高科技加密（骗你的，我自己打的星号
```

这样以来服务就启动起来了，可以访问`http://127.0.0.1:15672`看到RabbitMQ的web登录页面了。

> `docker-compose`并不是docker的一部分，而是一个用Python编写的docker编排工具。如果电脑上的话可以使用`pip install docker-compose`来安装它

# 0X01 使用Python调用RabbitMQ

[RabbitMQ的官方文档上有一个非常简单明了的介绍如何使用Python接入RabbitMQ](<https://www.rabbitmq.com/tutorials/tutorial-one-python.html>)。有两坨代码，一坨是sender另一坨是receiver，首先是sender：

```python
    #!/usr/bin/env python
    import pika

    # 建立和RabbitMQ的连接
    credentials = pika.PlainCredentials('shawn', '********')   # 两个参数：用户名和密码
    connection = pika.BlockingConnection(
        pika.ConnectionParameters('localhost', 5672, '/', credentials)  # 四个参数：机器、端口、虚拟主机（新手先不管它）、认证信息
    )
    channel = connection.channel()

    # 选择使用一个队列
    channel.queue_declare(queue='hello')

    # 发送一个消息
    channel.basic_publish(exchange='', routing_key='hello', body='Hello World!')
    print(" [x] Sent 'Hello World!'")

    # 断开连接
    connection.close()
```

然后是差不太多的receiver的这一坨

```python
    #!/usr/bin/env python
    import pika

    # 完全相同的建立连接
    credentials = pika.PlainCredentials('shawn', '********')
    connection = pika.BlockingConnection(
        pika.ConnectionParameters('localhost', 5672, '/', credentials)
    )
    channel = connection.channel()

    # 完全相同的选择使用一个队列
    channel.queue_declare(queue='hello')


    # 创建一个回调方法
    def callback(ch, method, properties, body):
        print(" [x] Received %r" % body)


    # 从'hello'队列来的消息交给`callback`方法处理
    channel.basic_consume(queue='hello', on_message_callback=callback, auto_ack=True)

    # 开始等待消息
    print(' [*] Waiting for messages. To exit press CTRL+C')
    channel.start_consuming()
```

现在就可以用`python sender.py`把消息塞到RabbitMQ中，再用`python receiver.py`拿到RabbitMQ队列中的消息了。

# 0X02 使用Django调用RabbitMQ（Celery）

Django在生产环境中经常需要Celery的支持，在Django中使用Celery主要是为了两大特性：定时任务和异步任务。如果够骚的话定时任务可以通过Linux的`crontab`来替代，但是异步任务目前还不太好离开Celery。通常部署Celery的时候后端都是Redis，这次可以尝试一下使用RabbitMQ（Celery默认就是支持使用RabbitMQ这类MQ的）。在正常配置了Redis作为后端的情况下切换到RabbitMQ其实是不麻烦的：唯一要做的就是将本来的`BROKER_URL`改成`BROKER_URL='amqp://shawn:********@localhost:5672//'`就可以了。

相比Redis，RabbitMQ自带web界面，可以方便的查看后台任务；而且作为broker来说性能更强劲。可能这也是Celery官方建议使用RabbitMQ的原因吧。

> Django3其实已经开始支持异步了，但等到大规模高质量应用可能还需要一段时间
