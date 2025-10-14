---
title: "Django 中事务的三种简单用法"
slug: "django-transaction"
date: "2018-07-10T15:24:00+0000"
lastmod: "2025-01-16T08:27:23+0000"
draft: false
tags:
  - "Django"
  - "ORM"
visibility: "public"
---
# 0X00 什么是“事务”

“事务”简单的说就是把一些数据库操作打包起来，要么就全部执行要么就全部不执行。

假设有一个操作是新建一个学生信息，有多张表分别记录了“基本信息、家庭信息、学校信息等“，那么就需要分成多步来新增这个学生的信息。但是如果在添加了”基本信息和家庭信息“两张表的内容后在添加”学校信息“时出现了错误那么数据库中就会存在该学生部分信息，从而使得数据库中的数据出现错误。

如果将这些操作放到一个”事务“中执行就可以在中途出现错误的时候所有数据库操作都不生效，当顺利执行完成之后使所有数据库操作都生效。总的来说就是”事务中出现错误则所有数据库操作都不生效，否则所有数据库操作均生效“。

具体可参见下面几个链接
[Database transaction - Wikipedia](<https://en.wikipedia.org/wiki/Database_transaction>)
[MySQL Document](<https://dev.mysql.com/doc/refman/8.0/en/commit.html>)

# 0X01 将事务绑定到http请求

在Django中实现数据库事务最简单明了的方法就是在数据库的配置中添加一个`ATOMIC_REQUESTS=True`。添加这个操作后Django会把每一个请求到响应的过程视为一个”事务“，从而在请求获得正确响应时应用这些数据库操作。如果在处理请求的过程中抛出了异常那么Django就会回滚这次事务。

***** 注意： ** 此时的所有处理的所有请求都会被包装成事务！

如果需要声明某些view不需要使用事务，那么可以通过使用如下方法阻止view使用事务

```python
    from django.db import transaction

    @transaction.non_atomic_requests
    def my_view(request):
        do_stuff()
```

而且事务会对数据库和Django造成更大的压力，所以此种方式并不是最好的使用事务的方法。

# 0X02 使用`@transaction.atomic`装饰器

我们使用事务通常是明确知道哪些方法需要使用，而不是像上面那种直接给所有view添加事务再一一排除那些不需要的。

所以Django中也有一种直接给某个方法标记为使用事务的方法：使用`@transaction.atomic`装饰器。例如下面的代码，如果你只需要给某个view使用事务或者只是需要给某个其他的方法使用事务，那么就可以使用这种方法来装饰function，从而使其在一个事务中。

```python
    from django.db import transaction

    @transaction.atomic
    def my_view(request):
        do_stuff()

    @transaction.atomic
    def some_func():
        do_stuff()
```

# 0X03 使用`with transaction.atomic:`语句

使用`@transaction.atomic`装饰器是直接给某个function打包为一个事务，但是在某些情况下还会有更小粒度的事务需求，下面给出一个情况。

```python
    from django.db import transaction

    from my_project import models


    def some_func():
        models.AAAAA.objects.filter().update(aaa='456')
        models.BBBBB.objects.filter().update(bbb='789')
        models.CCCCC.objects.filter().update(ccc='123')
        print '江XXX，亦XXX'
        '........'
```

如果这段代码在更新model：`BBBBB`的时候出现了错误，那么其实AAAAA是更新了的，且由于抛出了异常所以BBBBB和CCCCC都没有被更新。此时数据库中的数据就是不符合我们要求的。这个时候就可以使用`with transaction.atomic`的方法来将一个代码块打包为一个事务。

```python
    from django.db import transaction

    from my_project import models


    def some_func():
        with transaction.atomic:
            models.AAAAA.objects.filter().update(aaa='456')
            models.BBBBB.objects.filter().update(bbb='789')
            models.CCCCC.objects.filter().update(ccc='123')
        print '江XXX，亦XXX'
        '........'
```

此时AAAAA/BBBBB/CCCCC这三个model的操作已经”同生同死“了。

> 参考自： [Django中文文档](<https://yiyibooks.cn/xx/Django_1.11.6/topics/db/transactions.html>)
