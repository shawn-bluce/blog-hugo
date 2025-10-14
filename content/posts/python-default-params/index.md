---
title: "关于 Python 函数默认值的小问题"
slug: "python-default-params"
date: "2018-08-27T14:50:00+0000"
lastmod: "2025-01-16T10:01:31+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
> Python一切皆对象

# 0X00 困扰我的一个问题

前两天在工作上遇到了个问题，说来很简单：我要在每天的固定时刻统计系统中当天产生的一些数据并且用邮件发送给指定的人，又考虑到了程序的可复用性(统计其他日期)我并没有把参数写死，而是将其默认为当天的日期并可以指定参数。很容易我就写出了类似下面的代码。Ps.伪代码，不要过分纠结。

```python
    def export_statistic(export_date=datetime.date.today()):
        result = get_statistic_for_day(export_date)
        sendmail('今日数据统计结果', result, receiver_list)
```

并且将其配置在Celery中，每晚执行，并且在得到了第一天的正确数据后默认程序正确了。第二晚虽然收到了统计数据的邮件，但是发现日期是前一天的。以为是Celery或是服务器时间同步问题或是缓存等导致的，但是在多次检查后没有发现这个问题的根本原因。故临时使用`crontab`去执行这个定时任务，但这并不是长久之计。

# 0X01 到底发生了什么

纠结问题所在的时候突然想到“会不会函数的默认值在函数初次初始化的时候生成好就不再变了？”故而使用下面这段代码来检验自己的推断。

```python
    #/usr/bin/env python
    # coding=utf-8

    import time
    from datetime import datetime


    def test(date=datetime.now()):
        print date


    if __name__ == '__main__':
        for i in range(10):
            test()
            time.sleep(1)
```

果然输出的结果和我以前的设想不同，按照我以前的想法应该是输出的几个时间间隔为1s，但是结果却是每一行都相同（果然我的1s不见了）。

```
    2018-08-27 23:04:53.008236
    2018-08-27 23:04:53.008236
    2018-08-27 23:04:53.008236
    2018-08-27 23:04:53.008236
    2018-08-27 23:04:53.008236
    2018-08-27 23:04:53.008236
    2018-08-27 23:04:53.008236
    2018-08-27 23:04:53.008236
    2018-08-27 23:04:53.008236
    2018-08-27 23:04:53.008236
```

# 0X02 那是为什么呢

**Python一切皆对象** 这句话看来真的不是说着玩儿的。其实Python中的一个函数也是一个对象，而对象就会有初始化的时候。Python的函数在作为对象进行初始化的时候就计算了“默认参数”。比如上面的例子，在函数`def test(date=datetime.now())`初始化的时候就已经计算了`date=datetime.now()`为`2018-08-27 23:04:53.008326`，所以以后每次调用的函数test默认参数都是这个值，有一个经典的案例可以参考。

```python
    #!/usr/bin/env python
    # coding=utf-8

    def test(a, b=[]):
        b.append(a)
        print b


    if __name__ == '__main__':
        for i in range(10):
            test(i)
```

按照我之前的想法，输出的应该是`[0], [1], [2]...`这种，每次列表中只有一个元素，然而事实上是这样的。

```
    [0]
    [0, 1]
    [0, 1, 2]
    [0, 1, 2, 3]
    [0, 1, 2, 3, 4]
    [0, 1, 2, 3, 4, 5]
    [0, 1, 2, 3, 4, 5, 6]
    [0, 1, 2, 3, 4, 5, 6, 7]
    [0, 1, 2, 3, 4, 5, 6, 7, 8]
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

我们来通过`id()`来检查一下原因。Python中的`id(foo)`会返回foo在内存中的唯一标识，稍作修改把代码改成如下

```python
    #!/usr/bin/env python
    # coding=utf-8

    def test(a, b=[]):
        b.append(a)
        print id(b), b  # 只是在这里新增了id(b)的输出


    if __name__ == '__main__':
        for i in range(10):
            test(i)
```

我们可以看到，其实`b=[]`只执行了一次，每次使用的b并不是我们臆想中重新初始化的，而是一个现有的。

# 0X03 再次证实

自己写了几行代码来简单证明这个事情，按照我之前的臆想这个程序应该是看不到print输出的，因为看起来`hello()`并没有被调用到，但是其实呢？

```python
    #!/usr/bin/env python
    # coding=utf-8

    def hello(a='hello'):
        print a
        return a

    def world(b=hello()):
        return b


    if __name__ == '__main__':
        pass
```

解释器在执行`def hello(a='hello')`的时候正常下去了，生成了一个`hello()`方法的函数对象，但是在执行到`def world(b=hello())`的时候也要将`world()`初始化为一个固定的对象，那么就势必执行了`b=hello()`。所以我们可以看到最终还是会有一个`hello`的输出。

> 1s不见了当然不是因为时间被转移走了-_-

> 文章参考自[Python进阶-函数默认参数 珞樱缤纷-cnblogs](<https://www.cnblogs.com/crazyrunning/p/6867849.html>)
