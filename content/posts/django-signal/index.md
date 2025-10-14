---
title: "Django信号初级"
slug: "django-signal"
date: "2019-11-06T13:33:00+0000"
lastmod: "2025-01-16T08:24:26+0000"
draft: false
tags:
  - "Django"
  - "Python"
  - "Signal"
visibility: "public"
---
# 0X00 前言

实话讲，Django的信号(signal)机制其实用到的时候并不多，但是某些特定场景下一个信号能解决非常大的问题，所以信号这个东西还是值得了解一下的。那么为什么这里只说一些初级内容呢，主要是因为通过调查发现信号的高级知识用（我）的（也）很（不）少（会）。

目前我工作中用到的信号机制也比较少，所以可能有些事情说不到点上还请见谅。那我们开始吧~

# 0X01 什么是信号

“信号机制”光从名字上来就大概能懂了。应该就是：某人发出某信号，某人接受到之后做一些事情。所以看起来非常类似我们熟悉的“订阅-发布”模式，实际上也也确实很类似。整个信号机制分成这么几个部分：发布者、信号、接收者和一个函数。

我们传统战场上的一个行为来类比会比较好解释

  1. 发送者：类似于战场上打信号弹的人
  2. 信号：信号弹（信号弹会分成几种比如红色、蓝色、绿色的）
  3. 接受者：各个不同阵地都有人观察着战场的信号弹
  4. 一个函数：接受者看到信号弹后会对应作出战术动作

比如我们有三个人：小明、小强和李铁蛋，每人又有三种信号弹：红色、蓝色和绿色，又有三个不同阵地：路口、广场和理发店。其中每个人的信号弹不同，小明的红色信号弹打出去是一个”明“字，小强的是”强“，李铁蛋的是个”蛋“。

那么这个时候场上的小明发射了蓝色的信号弹（发送者发送了特定信号），三个阵地的人都看到了，但是之前首长说只有广场的阵地要响应小明的蓝色信号弹（提前固定好接受者要接收哪部分信号），广场的阵地接受到信号之后按照之前的计划前去攻打碉堡（接受者收到指定信号后执行一个函数）。

大概的流程是这个样子的，中间可能有些不准确不过大体是对的，下面我们来看一下Django自身内置的一些信号。

# 0X02 Django内置的信号

Django中的信号分两类：内置和自定义的。我们先来列出现有的部分信号：（更完整的Django内置信号可以看[官方文档](<https://docs.djangoproject.com/en/2.2/topics/signals/>)）

```python
    # model部分
    from django.db.models.signals import pre_init   # 数据模型构造前触发
    from django.db.models.signals import post_init  # 数据模型构造厚触发
    from django.db.models.signals import pre_save   # 数据对象保存前触发  instance.save()
    from django.db.models.signals import post_save  # 数据对象保存厚触发
    from django.db.models.signals import pre_delete # 数据对象删除前触发
    from django.db.models.signals import post_delete    # 数据对象删除厚触发

    # migrate部分
    from django.db.models.signals import pre_migrate    # migrate前触发
    from django.db.models.signals import post_migrate   # migrate后触发

    # request部分
    from django.core.signals import request_finished    # 请求结束后触发
    from django.core.signals import request_started     # 请求开始前触发
    from django.core.signals import got_request_exception   # 请求异常后触发
```

# 0X03 使用Django信号

使用一个Django信号首先要导入（或者编写）signal，接下来再设置好接受者，紧接着写好要执行的函数，然后注册它，最后调用就好了。

首先个人建议给signal单独放一个文件，如果你signal用的很多很多那就可以给每个app安排一个`signal.py`，或者如果你用的不过的话整个项目用一个`signal.py`就好了。首先我来在我的项目的`School/`下搞一个`signal.py`并写上下面的内容:

```python
    from django.dispatch import receiver
    from django.db.models.signals import pre_save

    from School.models import Student

    @receiver(pre_save, sender=Student)
    def print_hello_when_student_pre_save(sender, instance, **kwargs):
        print('hello,world')
```

> 装饰器参数中：第一个参数是信号类型，第二个参数是发送者；自定义的函数中instance就是数据对象了，但是由于是`pre_save`参数所以此时的instance还没有被真正写入到数据库中，所以如果打印`instance.id`的话其实是为空的

然后我们来注册这个，看一下现在`School/apps.py`的内容是这样的：

```python
    from django.apps import AppConfig


    class SchoolConfig(AppConfig):
        name = 'School'
```

我们稍加改动就能完成注册：

```python
    from django.apps import AppConfig


    class SchoolConfig(AppConfig):
        name = 'School'

        def ready(self):    # 这里的ready在Django启动的时候会自动被执行，然后我们的注册就完成了（并没有其实）
            from .signal import print_hello_when_student_pre_save
```

但是现在其实还不能用，我们知道在`settings.py`中我们只需要在`INSTALLED_APPS`中添加一个app名就可以了，但是我们这次需要把它改成`apps.py`下的app类名。我之前写的是`School`现在就要改成`School.apps.SchoolConfig`才行。（现在才是真的好了）

```python
    In [1]: from School.models import Student

    In [2]: Student.objects.create(name='shawn')
    hello,world
    Out[2]: <Student: Student object (1120)>
```

我们可以看到在我`create`一个数据对象的时候确确实输出了我们指定的`hello,world`。

> 关于`pre_save/post_save`的提示当使用`obj = models.Student.objects.first(); obj.name = 'hello'; obj.save()`的方法更新数据时会触发该信号，但是如果是用`models.Student.objects.filter().update(name='hello')`是不能触发的；在`pre_save/post_save`的响应函数里切忌再执行该model的`.save()`否则会进入死循环。

# 0X04 自定义信号

要使用自定义信号的时候并不是很多，不过还是可以说一下。我们拿上面信号弹的例子看一下

首先在目录下搞一个`custom_signals.py`文件，里面写好下面的内容

```python
    import django.dispatch

    # 声明一个信号：还接收两个参数（好像比信号弹强一些哈哈哈）
    blue_signal = django.dispatch.Signal(providing_args=['attack_time', 'attack_zone'])

    def callback(sender, attack_time, attack_zone, **kwargs):
        if sender == '小明':    # 只相应小明的信号
            print('attack it from {} {}'.format(attack_time, attack_zone))
        else:
            pass

    blue_signal.connect(callback)
```

然后我们启动shell来试试`python manage.py shell`，在shell里模拟一下发送信号。这个过程在正常程序中是写在`view`中的，此处只是为了展示方便

```python
    In [1]: from School.custom_signal import blue_signal    # 首先导入我们的信号

    In [2]: blue_signal.send(sender='李铁蛋', attack_time='now', attack_zone='left')     # 发送信号
    Out[2]:     # 并没有输出，因为接受者并不相应铁蛋的信号
    [(<function School.custom_signal.callback(sender, attack_time, attack_zone, **kwargs)>,
      None)]

    In [3]: blue_signal.send(sender='小明', attack_time='now', attack_zone='left')
    attack it from now left     # 小明的信号得到了相应，并且对应的参数也成功传进去了
    Out[3]:
    [(<function School.custom_signal.callback(sender, attack_time, attack_zone, **kwargs)>,
      None)]
```

# 0X05 结尾

Django的信号机制大概就是这个样子，我这里再贴出几个不错的参考资料吧

  1. [官方文档： Django signal](<https://docs.djangoproject.com/en/2.2/topics/signals/>)
  2. [Django的信号机制 卡瓦邦噶！](<https://www.kawabangga.com/posts/1997>)
  3. [django 信号（signal） 2BiTT](<https://www.cnblogs.com/qwj-sysu/p/4224805.html>)
