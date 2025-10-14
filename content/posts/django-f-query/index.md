---
title: "Django 中的 F()"
slug: "django-f-query"
date: "2018-08-29T14:16:00+0000"
lastmod: "2025-01-16T07:44:24+0000"
draft: false
tags:
  - "Django"
  - "ORM"
  - "Python"
visibility: "public"
---
# 0X00 内容比较少，不分标题

我们对Django中的model进行查询时通常是**某个字段和一个常量** 对比，比如下面这种写法

```python
    Student.objects.filter(name='shawn')
    Student.objects.filter(age=233)
    Student.objects.filter(gender__in=('F', 'M'))
```

如果遇到高级的查询可能会使用`Q()`查询，不过也只是进行多个条件的查询

```python
    Student.objects.filter(
        Q(name='shawn') | Q(gender='M')
    )
    Student.objects.filter(
        Q(gender='F') | ~Q(age=233)
    )
```

> [这里是我的另一篇介绍Q()的博文。](<https://blog.just666.cn/2018/05/20/django-query-q/>)

但是如果有这样一个需求：”查询订单中结束时间和开始时间的间隔大于45分钟的“。那应该怎么办的？因为订单的开始时间和结束时间都是一个字段，我们需要对比同一条数据中的两个字段。这时候可以使用`F()`来查询。

```python
    import datetime

    from django.db.models import F
    from my_project.models.Order

    forty_five_minutes = datetime.timedelta(minutes=45)
    Order.objects.filter(end_time__gt=F('start_time') + forty_five_minutes) # 查询结束时间大于开始时间加上45分钟的订单
    Order.objects.filter(operator__age__lte=F('client__age'))   # 查询订单操作员年龄小于等于客户的（我也不知道有啥用，例子而已）

    one_second = datetime.timedelta(seconds=1)
    OldMan.objects.filter(age__gt=F('age') + one_second)    # 查询年龄比自己年龄多1S的大人？哈哈哈
```

> 本段内容的官方文档：<https://docs.djangoproject.com/en/2.1/ref/models/expressions/#f-expressions>
> 本段内容的另一篇博客：<https://www.cnblogs.com/liuq/p/5946803.html>
