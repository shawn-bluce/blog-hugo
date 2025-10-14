---
title: "使用 Django 中的 select_related 和 prefetch_related 优化查询"
slug: "django-select_related-prefetch_related"
date: "2019-05-12T13:59:00+0000"
lastmod: "2025-01-16T07:42:08+0000"
draft: false
tags:
  - "Python"
  - "Django"
  - "ORM"
visibility: "public"
---
# 0X00 前言

没有什么前言，只有一个数据库模型，下面的代码使用这个模型拿来测试。

```python
    from django.db import models


    class Major(models.Model):
        name = models.CharField(
            '名字',
            max_length=100,
            blank=True,
            null=True,
        )


    class Student(models.Model):
        name = models.CharField(
            '名字',
            max_length=100,
            blank=True,
            null=True,
        )
        select_major = models.ForeignKey(
            Major,
            verbose_name='专业',
            on_delete=models.SET_NULL,
            blank=True,
            null=True,
            related_name='main_student',
        )
        ext_major = models.ManyToManyField(
            Major,
            verbose_name='附加专业',
            related_name='ext_student',
        )
```


我们先假设Major表存在10条数据，而Student表存在1万条数据。

# 0X01 使用select_related

如果我们要得到所有的学生和他们所学专业的名字，那么我们可以轻松写出下面的代码

```python
    for student in Student.objects.all():
        print(student.name, '-->', student.selected_major.name)
```

这样就能得到所有学生姓名和他们所学的专业名了，但是重点在于这次查询其实是一个很低效的查询，因为在`Student.objects.all()`的时候查询了一次数据库，而且每次访问`student.selected_major.name`的时候都会再查询一次数据库，基于上述条件这两行代码将会查询10001次数据库，是一个比较夸张的数字了。那么如何用`select_related`来优化这次查询呢？

```python
    for student in Student.objects.all().select_related('major'):
        print(student.name, '-->', student.selected_major.name)
```

其实就是在`all()`之后添加了`select_related('major')`，这次就只需要对数据库进行一次查询。在我本地的类似环境下测试结果是不使用`select_related`消耗的时间是优化后的400%左右。

> 本质上是`select_related`进行了数据库级的JOIN操作，具体的大家可以通过查看`print(Model.objects.filter().query)`或者`django-extensions`等方法查看具体的SQL

这里可能会有一种声音“查询从10001次到1次差了几乎10000倍时间却只省下了70%多？”这个问题其实比较好理解，JOIN操作本来就会使一次SQL查询变的很慢，毕竟要跨越多张表。

`select_related`是用于优化“多对一”结构中从“多”表出发查“一”表，也就是这里的多个学生对一个专业，从学生出发查询得到“专业”。

这里我贴出使用前和使用后的两坨SQL，大家可以对比一下。（这些SQL都是Django-extensions分析得出的）

```sql
    优化前
    SELECT "School_student"."id",
           "School_student"."name",
           "School_student"."select_major_id"
      FROM "School_student"
     LIMIT 3

    SELECT "School_major"."id",
           "School_major"."name"
      FROM "School_major"
     WHERE "School_major"."id" = 4

    SELECT "School_major"."id",
           "School_major"."name"
      FROM "School_major"
     WHERE "School_major"."id" = 5
    一直重复，直到循环结束


    优化后
    SELECT "School_student"."id",
           "School_student"."name",
           "School_student"."select_major_id",
           "School_major"."id",
           "School_major"."name"
      FROM "School_student"
      LEFT OUTER JOIN "School_major"
        ON ("School_student"."select_major_id" = "School_major"."id")
    只剩下了一条使用了JOIN的SQL，显然会比上面快很多
```

# 0X02 使用prefetch_related

`prefetch_related`是从“一对多”结构中“一”表出发查“多”表，也就是说从专业表出发查询得到学生信息。比如我们想看这所有专业中每个专业下的学生

```python
    for major in Major.objects.filter():
        print(major.main_student.all())
```

这种查询是最容易写出来的，不过需要注意的一点是，这里第一行循环前有一个`filter()`会查询一次数据库，后面每一次`main_student.all()`都会再查询数据库。我们只有10个专业，查11次数据库还行问题不大，但是随着数据增多这里查询数据库的次数会呈线性增长。那么如何解决这个问题呢？

```python
    for major in Major.objects.filter().prefetch_related('main_student'):
        print(major.main_student.all())
```

就只是向上面使用`select_related`一样添加一个`prefetch_related`在`filter()`后面就可以了。修改了之后的代码只会查询两次数据库：第一次把所有的专业查出来了，也就是`Major.objects.filter()`的作用；第二次是使用一个`SELECT * FROM student WHERE select_major_id IN (x,x,x,x,x,x)`形式的SQL查询到了对应的Student。**然后使用Python将其组装整合而非数据库** ，这样就能大幅度减少查询数据库的次数了。

这里我贴出使用前和使用后的两坨SQL，大家可以对比一下。（这些SQL都是Django-extensions分析得出的）

```sql
    优化前
    SELECT "School_major"."id",
           "School_major"."name"
      FROM "School_major"

    SELECT "School_student"."id",
           "School_student"."name",
           "School_student"."select_major_id"
      FROM "School_student"
     WHERE "School_student"."select_major_id" = 4
     LIMIT 21

    SELECT "School_student"."id",
           "School_student"."name",
           "School_student"."select_major_id"
      FROM "School_student"
     WHERE "School_student"."select_major_id" = 5
     LIMIT 21
    ......一直重复直到循环完所有的


    优化后
    SELECT "School_major"."id",
           "School_major"."name"
      FROM "School_major"


    SELECT "School_student"."id",
           "School_student"."name",
           "School_student"."select_major_id"
      FROM "School_student"
     WHERE "School_student"."select_major_id" IN (4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100, 101, 102, 103)
```

# 0X03 总结

还有总结？大家都有，我也有好了

通常来说恰当的使用`select_related`和`prefetch_related`可以大幅度提升自己ORM查询的速度，很多时候大家只是写了能用的查询，现在可以尝试着使用`select_related/prefetch_related`写出好用的查询啦
