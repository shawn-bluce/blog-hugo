---
title: "Django 与 Django REST framework 中的这些 \"空\""
slug: "django-drf-null-and-blank"
date: "2018-05-23T15:55:00+0000"
lastmod: "2025-01-16T08:25:22+0000"
draft: false
tags:
  - "Django"
  - "Python"
  - "DRF"
  - "ORM"
visibility: "public"
---
# 0X00 Django Model中的空

Django的Model常见两个与空有关的参数：`null`和`blank`。其中`null`是数据库层面的是否允许为Null，而`blank`则将空处理为空值。比如一个`CharField`的`blank=True`，那么这个字段在没有赋值的情况下入库，这个字段就会是空字符串而不是Null。

```python
    In [13]: s = Student()

    In [14]: s.age=1

    In [15]: s.save()

    In [16]: s.name
    Out[16]: u''
```

如果将`blank=False`再不赋值该字段进行保存则入库的就是`Null`。

```python
    In [3]: s = Student()

    In [4]: s.age = 1

    In [5]: s.save()

    In [6]: s.name

    In [7]: type(s.name)
    Out[7]: NoneType
```

所以换句话说，`null=True`是数据库层面允许存储Null，而`blank=True`则是允许存入"空字符串"等表示空的值。

# 0X01 Django REST framework中的空

在Django REST framework的serializer中的字段，有三个与空有关的，都是在创建或更新中生效。分别是`allow_blank/allow_null/require`这三个。其中`allow_blank=True`表示着`CharField/ListField`等允许传入`""/[]`等空值；`allow_null=True`表示着允许传入`{"name": null, "age": null}`这种`null`空；`require=True`则表示着字段必填，如果name字段被设定了`require=True`那么在`POST/PUT/PATCH`等创建或更新数据时这个字段是必须要填写的。
