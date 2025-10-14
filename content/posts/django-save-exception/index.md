---
title: "记一次 Django save 导致的数据异常"
slug: "django-save-exception"
date: "2020-07-11T05:22:00+0000"
lastmod: "2025-01-16T08:15:35+0000"
draft: false
tags:
  - "Django"
  - "ORM"
visibility: "public"
---
# 0X00 按惯例得有一个标题

众所周知`save`是 Django 中最常用的保存数据的方法。但是一般来说大家经常会把“常用“理解成“万能“，然后能用的时候就全用这一种方式。不过编程 中是没有所谓的“一招鲜吃遍天“的，Django 之所以提供了那么多中保存数据的方法也侧面证实了这一点。

首先来看一下我遇到的这个问题：

```python
    from .models import Student
    from .utils import calculate_score

    queryset = Student.objects.all()
    for student in queryset:
        student.score = calculate_score(student)    # 调用一个工具函数计算该学生的成绩
        student.save()
```


这段代码乍一看没有什么问题，因为计算的值是通过`calculate_score`这个函数进行的，所以不能使用`queryset.update(xxx)`的方法。然后咱们看一下 Django 文档是如何描述 queryset 的。

> QuerySets are lazy – the act of creating a QuerySet doesn’t involve any database activity. You can stack filters together all day long, and Django won’t actually run the query until the QuerySet is evaluated.
>
> [QuerySets are lazy](<https://docs.djangoproject.com/en/2.2/topics/db/queries/#querysets-are-lazy>)

> nternally, a QuerySet can be constructed, filtered, sliced, and generally passed around without actually hitting the database. No database activity actually occurs until you do something to evaluate the queryset.
>
> [When QuerySets are evaluated](<https://docs.djangoproject.com/en/2.2/ref/models/querysets/#when-querysets-are-evaluated>)

QuerySets are lazy 内容总结来说就是“Django 中的 QuerySet 只有在用的时候才会真的去数据库里查，而不是生成 QuerySet 的时候“。后面的 When QuerySet are evaluated 则标明了什么叫做“真正在使用“，给出了下面几个条件，当你做这些事情的时候就是“真正在使用“了。这些条件包括：迭代、 切片、列表等（我英文水平小学三年级，解读地不对的地方还希望大家指出）。 所以显然我们对这个 queryset 来了个 `for` 循环就满足了上述的“迭代“，所以这时候数 Django 就会真正的从数据库中将数据真正的 **取出来** 。

现在问题来了。我们思考一个问题，如果我一秒钟能计算10 个学生的成绩，然后整个`Student`表有 3W 学生，得出“处理所有学生信息需要消耗 50 分钟的时间“这样的结论（每秒 10 条和一共 3W 是乱写的，真实数据通常比这个大得多）。

**如果在执行这个循环的时候，某位同学修改了自己的的信息，比如手机号，会发生什么？** 有两种可能：第一种可能是这位同学修改自己手机号的时候计算分数的循环已经把他的分数计算完了，那么他的手机号修改也生效了（这种最好）。但是如果他改手机号的时候循环还没到他呢？假设他把手机号从原来的 123 改成了 456，那么他改完手机号的一瞬间数据库里存进去的确实是 456，没有问题。**但是 queryset 里是他改手机号以前取出来的 123** ，这时候循环到他了，计算完之后来了一个`student.save()`，如此一来他刚刚改好的手机号码就又回到了 123。

所以说这种写法并不会 100% 出现问题。整个循环耗时越久，出现问题的可能性越大；系统中数据变更越频繁，出现问题的可能性越大。当然了，bug 就是 bug，不能因为 bug 没触发就无所畏惧了，还是得解决的。通常来说有两种解决方法，下面是第一种

```python
    for student in queryset:
        new_score = calculate_score(student)
        Student.objects.filter(id=student.id).update(score=new_score)
```

这种方式仍然比较 young 比较 simple 比较 naive，不过**又不是不能用** 。

![](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/20200711151128.jpeg)

但是这种用法显然是不好的，而且 `update` 本来也不是让我们这么用的。所以我们还是得回到`save`上，Django 其实已经提供了一个参数给 `save` 了，可以用下面这种方法

```python
    for student in queryset:
        student.score = calculate_score(student)
        student.save(update_fields=['score'])
```

也就是在 `save` 的时候带上具体需要更新哪个字段，其他的就不更新了。而且通过传递的参数也可以看出，指定的是多个字段，如果有需要修改多个字段的话，就只修改这一个就好了。

不过其实这里还是有一个潜在问题的，那就是说：恰好我们在更新 score 的时候，其他地方也在更新这个。不过这个更多的时候就是我们程序逻辑的问题了，因为在几乎同一时间对一个字段进行修改，然后修改的双方又互相不知道的话，总是会出问题的。
