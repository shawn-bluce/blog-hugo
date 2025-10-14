---
title: "使用 Django 中的 Q 对象查询"
slug: "django-q-query"
date: "2018-05-20T07:37:00+0000"
lastmod: "2025-01-16T07:51:31+0000"
draft: false
tags:
  - "Django"
  - "Python"
  - "ORM"
visibility: "public"
---
# 0X00 普通的查询

```python
    from django.db.models import Q

    queryset.filter(Q(age=233)) # 找到233岁的人
    queryset.filter(Q(name='shawn')) # 找到名为shawn的人
```

这种查询方式与普通的方式比起来没什么区别。

```python
    queryset.filter(age=233)
    queryset.filter(name='shawn')
```

# 0X01 AND

```python
    from django.db.models import Q

    queryset.filter(Q(age__range=(18, 25), Q(gender='F'), Q(beautiful=True)) # 找到18到25岁的漂亮女生
```

把多个条件用逗号分割开就可以了，或者使用`&`符分割开。

# 0X02 OR

```python
    from django.db.models import Q

    queryset.filter(Q(gender='F') | Q(province=u'四川') | Q(name=u'王铁蛋'))
```

这里用`|`符号分割开筛选条件，最终筛选得到的是"所有女生、四川人和叫王铁蛋的人"，也就是说相当于分别筛选了这三个条件，最终取了并集。这种查询如果用普通方法进行查询就会很麻烦，可能要写成下面这样：

```python
    queryset.filter(gender='f') | queryset.filter(province=u'province=u'四川') | queryset.filter(name=u'王铁蛋')
```

# 0X03 NOT

使用Not查询的方式就比较诡异了

```python
    from django.db.models import Q

    queryset.filter(~Q(gender='M')) # 找到非男生
```

# 0X04 组合技

有的时候经常需要查询同样的条件多次，这种方法就可以一次编写查询条件多次执行

```python
    import operator
    from django.db.models import Q

    q1 = reduce(operator.and_, Q(gender='F', age__range=(18, 25)))
    q2 = reduce(operator.or_, Q(gender='M', age__range(3, 5)))

    queryset.filter(q1)
    queryset.filter(q2)
```