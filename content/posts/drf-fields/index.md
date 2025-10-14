---
title: "Django REST framework 中不那么常用的 Fields"
slug: "drf-fields"
date: "2018-10-13T06:26:00+0000"
lastmod: "2025-01-16T08:44:24+0000"
draft: false
tags:
  - "Django"
  - "ORM"
  - "DRF"
visibility: "public"
---
平时使用Django REST framework的时候除了常用的几个字段类型之外其实没有哪些字段是必须的了，不过了解一下这些「非必需」的字段能给日常的编程任务带来大幅度的效率提升呢。

# 0X00 EmailField

首先是`EmailField`，这个字段本质上是`CharField`但是单纯的添加了一个完善的校验，可以免去我们手工编写正则和对应报错信息的过程，简单地定义之后就可以使用了。

```python
    user_email = EmailFIeld(
      min_length=5,
      max_length=30,
      allow_blank=False,
    )
```

# 0X01 URLField

这里的`URLField`跟上面的`EmailField`类似，在`CharField`的本质上添加了对应的校验，使用时会校验该字段是否符合url的校验规则。

```python
    article_url = URLField(
      min_length=10,
      max_length=100,
      allow_blank=False,
    )
```

# 0X02 IPAddressField

`IPAddressField`也是一样，只是在`CharField`上添加校验。不过有一个针对ip的参数`protocol`，顾名思义，可以设定校验不同版本的ip地址。`protocol`参数的值从这三个中选一个`both/IPv4/IPv6`，其中默认选择的是`both`，也就是说同时会接受IPv4和IPv6两种地址。

```python
    source_ip4 = IPAddressField(
      required=True,
      protocol='IPv4',
    )

    source_ip6 = IPAddressField(
      required=True,
      protocol='IPv6',
    )
```

# 0X03 DecimalField 带有精度的

与`FloatField`不同的是，`DecimalField`会校验小数点位数，两个重要参数：`max_digits`数字总长度(不含小数点)，`decimal_places`小数点后几位。

```python
    # 不含小数点最长5位，精确到2位小数；也就是说最大为999.99
    num_a = DecimalField(
      max_digits=5,
      decimal_places=2,
    )

    # 不含小数点最长7位，精确到3位小数；也就是说最大为9999.999
    num_b = DecimalField(
      max_digits=7,
      decimal_places=3,
    )
```

# 0X04 DurationField

关于时间的字段用的最多的就是`DateTimeField/DateField/TimeField`了，但是这些都是时间点，而`DurationField`是时间段，就像Python中`datetime.timedelta`一样，使用的是`[DD] [HH:[MM:]]ss[.uuuuuu]`格式。

```python
    work_duration = DurationField(
      min_value=datetime.timedelta(days=12, hours=10, minutes=32, seconds=4, microseconds=3),
      max_value=datetime.timedelta(days=10, hours=1, minutes=2, seconds=4, microseconds=3),
    )
```

# 0X05 ListField

列表类型，自身是一个列表，需要单独指定`child`的类型。create/update的时候需要提供类似这样的参数`name_list = ['shawn', 'nwahs', 'shanw', 'sahwn']`。

```python
    email_list = ListField(
      min_length=3,         # 列表最少有3个元素
      max_length=100,       # 列表最多有100个元素
      child=EmailField(     # 列表中的元素为EmailField
        min_length=5,
        max_length=30,
        allow_blank=False,
      )
    )
```

# 0X06 DictField

与`ListField`很类似，`DictField`也需要单独指定`child`的类型，这里的child指的是dict的value，而key则默认为字符串。

```python
    email_list = DictField(
      child=EmailField(     # dict中的元素为EmailField
        min_length=5,
        max_length=30,
        allow_blank=False,
      )
    )
```

# 0X07 HiddenField

有些字段需要用，但是又不需要用户提供，比如说某条数据的更新时间应该在更新数据的时候自动设置为当前时间，而非用户传递的值。这时就可以使用这个`HiddenField`
字段。这个字段对用户来说是隐形的，只需要自己在serializer中设定好就可以。

```python
    update_time = HiddenField(default=datetime.datetime.now())
```

> 关于Django REST framework中Serializer fields更多的内容可以参考[官方文档](<https://www.django-rest-framework.org/api-guide/fields/#serializer-fields>)
