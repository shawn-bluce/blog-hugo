---
title: "Django 中的一些非入门级用法"
slug: "django-non_beginner-level-usage"
date: "2018-09-06T13:39:00+0000"
lastmod: "2025-01-16T08:12:07+0000"
draft: false
tags:
  - "Django"
  - "ORM"
visibility: "public"
---
为什么这里说是"非入门级"用法呢，因为我个人觉得这是我接触Django之后一段时间才开始了解的用法，但是说是高级用法又太夸张了，所以用了这么一个诡异的”非入门级“的定位。

下面的示例中使用下面的model，简单描述一下并非真实代码

```python
    from django.db import models

    class Staff(models.Model):
        name = models.CharField(max_length=10)
        age = models.IntegerField()


    class Order(models.Model):
        staff = models.ForeignKey(  # 订单负责人
            'Staff',
            null=True,
            on_delete=models.SET_NULL,
        )
        price = models.IntegerField()   # 订单的价值
```

# 0X00 使用Avg()/Sum()/Count()/Max()/Min()

这些方法的用法很简单，顾名思义。不过需要配合下面介绍的`annotate()`或`aggregate()`使用。

`from django.db.models import Avg, Sum, Count, Max, Min`

name | function
---|---
Avg | 求平均数
Sum | 求和
Count | 计数
Max | 求最大
Min | 求最小

# 0X01 使用annotate()

最基础的查询就是从一张表中查询符合某条件的字段，而使用`annotate()`可以得到一些我们手动计算得到的值，并将其作为Queryset中Item的一个属性来调用。

如果我想要查询每个人(Staff)手下有多少个订单(Order)，那么该怎么查呢，使用初级的用法可以写出类似下面的代码。但是有一个比较严肃的问题：会产生`用户数量 + 1`次的查询。这里只有少数用户问题不大，如果有上千甚至上万个用户，那么就会产生几千几万次查询，那对数据库的压力是很恐怖的。

```python
    staff_list = Staff.objects.filter()
    for staff in staff_list:
        order_count = Order.objects.filter(staff=staff).count()
```

使用`annotate()`方法就可以有效解决这个问题

```python
    staff_queryset = Staff.objects.filter().annotate(staff_order_count=Count('order'))
    for staff in staff_queryset:
        print(staff.staff_order_count)
```

这里的`staff_order_count`字段是表中并不存在的，是通过`annotate()`方法临时存储的一个字段。同样的，再一个`annotate()`方法中可以加入多个参数，使用同样的方法去统计和获取数据即可。

> 在`annotate()`中使用`Count()`一定要是有外键关联才行。例如本例中，Order表中有一个外键字段staff与表Staff关联起来了，那么就可以在Django中通过Staff.order_set来获取关联到Staff的Order，所以也就可以使用`Count()`方法来进行统计了。

生成查询的SQL语句也打出来方便理解。

```sql
    SELECT "Post_staff"."id", "Post_staff"."name", "Post_staff"."age", COUNT("Post_order"."id") AS "x" FROM "Post_staff" LEFT OUTER JOIN "Post_order" ON ("Post_staff"."id" = "Post_order"."staff_id") GROUP BY "Post_staff"."id", "Post_staff"."name", "Post_staff"."age"
```

# 0X02 使用aggregate()

使用`aggregate()`可以使得查询的返回值由一个`Queryset`变成一个`dict`，每个key和对应的value由自己计算得到。

如果我需要计算出所有人中年龄最大、最小、平均值该怎么办？初级用法可能需要先用一个查询得出所有人的年龄，然后再单独去计算最大最小平均值。写出类似如下代码，虽然目前问题不大，不过当逻辑复杂起来之后就会难以理解并且代码量较大。

```python
    queryset = Staff.objects.filter()
    age_list = [staff.age for staff in queryset]

    age_sum = sum(age_list)
    age_max = max(age_list)
    age_min = min(age_list)
    age_avg = (sum(age_list) * 1.0) / len(age_list)
```

但是如果使用`aggregate()`方法写出不仅逻辑清晰不易出错，而且代码量少了很多，更简单易读。

```python
    Staff.objects.filter().aggregate(
        age_sum=Sum('age'),
        age_max=Max('age'),
        age_min=Min('age'),
        age_avg=Avg('age'),
    )
```

这段代码就会直接输出如下dict，需要的数据直接取即可。

```python
    {
        "age_sum": 71,
        "age_max": 30,
        "age_min": 19,
        "age_avg": 23.666666666666668
    }
```

# 0X03 使用Case/When

Django中的`Case()/When()`是非常实用一对方法，恰当使用可以大幅度减小统计功能的代码量、逻辑复杂度等。

假设有如下需求”年龄小于18的为未成年(1)，年龄在19到30之间的为青年(2)，年龄在31 到60的为中年(3)，其他为老年(0)“，那么使用`Case/When`方法再配合`annotate()`方法就可以优雅得实现功能。

```python
    Staff.objects.filter().annotate(
        age_tag = Case(
            When(
                age__lt=18,
                then=1，
            )，
            When(
                age__lt=30,
                then=2,
            )
            When(
                age__lt=60,
                then=3,
            )
            default=0,
            output_field=IntegerField(),
        )
    )

    # 上面的代码类似于这种
    age_tag = 1 if age < 18 else 2 if age < 30 else 3 if age < 60 else 0

    # emm...这种更形象一点
    if age < 18:
        age_tag = 1
    elif age < 30:
        age_tag = 2
    elif age < 60:
        age_tag = 3
    else:
        age_tag = 0
```

上面是计算一个QueyrSet中每一个item的情况，还有一种情况是统计一个model中所有数据，例如这个需求：”统计所有Order中，单价最高、最低和平均值“。使用`Case()/When()`也可以完成任务，并且比初级用法更好一些。

```python
    Order.objects.filter().aggregate(
        max_price=Max('price'),
        min_price=Min('price'),
        avg_price=Avg('price'),
    )
```

> 内容整理的有点差，各位发现了什么疏漏和错误请及时联系我，防止误导别人。如果文章对大家带来了帮助，那我还是很开心的 嘿嘿
