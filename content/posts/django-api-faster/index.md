---
title: "如何让 Django API 再快一点"
slug: "django-api-faster"
date: "2020-05-13T16:15:00+0000"
lastmod: "2025-01-16T07:49:41+0000"
draft: false
tags:
  - "Django"
  - "Python"
visibility: "public"
---
# 0X00 前言

> 啊，这个破系统怎么这么慢。 --你写的程序的用户

是的，我用Django写的程序经常会出现性能问题，有时候是逻辑问题、有时候是数据库问题、有时候又是机器问题。我就现在这儿总结一波我自己的经验好了（这里都是基于我自己的经验来的，可能会相对比较简单，没有太骚太复杂太高级的东西）。这儿默认大家都是用的Django + Django REST framework了，因为我自己是用的这套技术栈，而且这套技术栈也算是Django生态下前后端分离的最常见的了。

# 0X01 问题出在哪儿呢

众所周知"想要解决问题，首先就要找到问题在哪儿"。那怎么判断问题在哪儿呢？

![htop截图](https://tva1.sinaimg.cn/large/007S8ZIlgy1ger78bhmijj32qg07s7b9.jpg)

  1. 首先登到服务器上看`htop`，有面有一个`Load average`就是综合负载。一般来说，如果你的服务器是n核心的，那负载在n-2以下就算是正常的，快到n了也不是不能用，但是就要考虑升级了。这里给出来了三个负载值，从左到右依次是1分钟、5分钟、10分钟的负载情况。（为什么说是一般情况呢，如果你就只有一两个核心，那这个算法肯定不生效；如果你有128核，那负载到126了就意味着马上就炸🤣。所以说一半双核心不超过1.5、4核不超过3，8核不超过6这种）
  2. 如果确定了就是机器性能的问题，那就好办了，升级服务器就好（当然不是不够久升级，还是要觉得当前的数据量啊并发啊已经挺高了再考虑。要不然一慢就升级服务器，那岂不是太奢侈了，而且对自己的代码质量也没有一点好处）
  3. 我们假设不需要升级服务器配置，那就从程序和数据库两个方面来说。一般是先打开MySQL的慢查询日志，然后根据慢查询日志来逐渐优化表结构，优化查询，优化程序逻辑。

# 0X02 代码质量低or逻辑问题

代码质量低是个问题，一般来说呈现在这几个地方：多余的循环次数、查了完全没卵用的数据、进行额外的操作。我们都知道计算机里几种处理速度的差距是巨大的`CPU缓存>>内存>>硬盘>>网络`，这四个之间的性能差异两两之间往往可以差出至少一个数量级（其实随着网络发展，现在网络速度已经可以赶上机械硬盘了）。而其中最慢的就是I/O了，所以我们应该尽一切可能避免I/O，而且有一点要注意的是"读写数据库"当然也算I/O。下面列举两种常见的问题

## 多余的I/O

```python
    for student_data in studnet_list:
        token = get_token_from_another_system_with_http_api()
        response = requests.post(url, student_data, headers={'token': token})
```

比如说这部分代码，我们都知道一个token不应该是一次性的，那把这个多余的取token的方法放在循环外面就好了。其实如果`get_token_from_another_system_with_http_api()`不是从其他的web服务上取token而是`get_token_from_local_cache()`的话，虽然也还是执行了多余的操作，但是就好得多了。

## 查了完全没卵用的东西

```python
    queryset = Student.objects.filter(age__gte=20, gender='F')
    for student in queryset:
        send_mail(studnet.email, '一个标题', 'hello,world')
```

这部分代码看起来问题不大，但是假设我们有10W的学生，并且`Student`表有大几十个字段，那到`for student in queryset`的时候，就会卡住一会儿（如果机器不太行的话可能会卡很久）。其实我们知道，默认这样的查询是`SELECT * FROM student WHERE xxxx`来的，把所有数据都取出来了。如果我们稍加改动

```python
    queryset = Student.objects.filter(age__gte=20, gender='F').only('email')
    for student in queryset:
        send_mail(studnet.email, '一个标题', 'hello,world')
```

仔细看，其实就只是在`filter()`后面加了`only('email')`，这就相当于是`SELECT email FROM student WHERE xxxxx`了，效率明显高了好多。或者直接改成

```python
    email_list = Student.objects.filter(age__gte=20, gender='F').values_list('email', flat=True)
    for email in email_list:
        send_mail(email, '一个标题', 'hello,world')
```

这样返回来的`queryset`里的元素就是email了。

# 0X03 数据库瓶颈（MySQL）

**数据库一定要多占内存，数据库一定要多占内存，数据库一定要多占内存。**一般来说，在Linux系统下是用多少内存分配多少，但是我们MySQL通常都是独立部署的，有且只有一个MySQL，所以就直接一次性给MySQL分配够内存，这是最好的方法。记得内存就是买来用的，买内存回来结果一年到头都是30%的占用，那岂不是亏了吗哈哈哈哈哈🤓

**数据库机器不要开swap，数据库机器不要开swap，数据库机器不要开swap** 。数据库的机器内存不够了，就加内存或者优化查询，万万不可使用交换分区。只要你的数据库机器一开swap分区，再结合上面的原则，就意味着**瞬间爆炸** 。因为内存一直都是几乎占满的情况，你一打开交换分区，Linux就会疯狂开始用内存和硬盘进行交换，本来你MySQL里有很多东西放内存里就是图个快的，结果又给在swap的机制下放回磁盘了，再折腾一圈下来甚至比直接在磁盘里还要慢。

## 索引问题

因为把优化查询改成了优化`ORM`，所以也就理所当然放到上一个段落里了，那这里就只剩下索引为题了。索引简单来说就是："针对某些字段，牺牲内存和写入速度换取查询速度"。所以说如果有些字段你很少改，甚至写进去就不会再变了，然后又要疯狂的以它为条件查询，那就给他加个索引。不过加索引之前有几个注意的点：

  1. 这个字段一定是读取频率远高于写入频率的；
  2. 这个字段的"唯一性"要高，比如学生表的身份证号这种，每条数据都有不同的身份证号；
  3. 这个字段要在SQL中的`WHERE`子句后面，而不是`SELECT`后面。也就是说：应该是条件，而非需要得到的数据；

具体的可以看我的[另一篇博客]](<https://blog.just666.com/2019/09/15/database-index/>)。

# 0X04 机器瓶颈（Linux）

机器瓶颈，这里给几种简单的排查、结解决方法，主要还是得靠运维同事了。

## htop

htop就是我们上面说到的看CPU/内存/进程和负载的工具，便于你找到疯狂消耗CPU或者内存的程序。

> 这个htop排序那里是可以用鼠标点的喔

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ger9ndnd8gj31gy0u0u0x.jpg)

## swap

如果你机器内存爆炸💥了，还不能第一时间加内存上去，那就只有先用交换分区缓一下。`swapon`和`swapoff`两个命令可以帮到你，具体的可以搜索一下，是可以在不停机的情况下加入新的交换分区和关闭交换分区。

一个奇技淫巧：可以使用`dd`命令搞一个块文件，然后格式化成`swap`格式，最后挂成交换分区喔。

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ger9v07ilqj31gy0u0twy.jpg)

## df/du

`df`可以看到当前挂载的磁盘，哪些快要满了。`du`可以方便得看目录的大小，还有一个`ncdu`是`du`的进阶版，是一个类似图形化的界面，用起来更舒服。能确认到哪个目录占用的空间多，然后指向性得清理一些数据。有时候磁盘满了都不知道是什么东西占了空间，这时候du和ncdu就很好用了。
