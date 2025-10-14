---
title: "Django 中的 objects"
slug: "django-objects"
date: "2020-07-14T12:55:00+0000"
lastmod: "2025-01-16T07:51:54+0000"
draft: false
tags:
  - "Django"
  - "Python"
  - "ORM"
visibility: "public"
---
# 0X00 objects 是个啥

想必所有用过 Django 的人都会用到 Django 自带的 ORM 进行数据库查询。那既然用过 Django 的 ORM 就来看一下这段代码好了， `models.Stuent.objects.filter(name='Shawn')` 这段代码是什么意思呢？很简单，就是查询到名字为"Shawn"的学生信息。具体来说， `models` 应该是一个放了多个 model 的文件，`Student` 是一个具体的模型，`filter` 是筛选，`name='Shawn'` 则是筛选条件。那么问题来了，中间那个 `objects` 是个啥呢？（你知道？知道还在这儿看啥，有这空看看其他文章，打打游戏看看电影不好吗🤣）

通过 `type` 可知，这个 `objects` 是一个 `django.db.models.manager.Manager` 的实例（或者是他子类的实例）。然后我们来看看这个`Manager`是个什么❓

![Django ORM](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/20200714212410.png)

> A Manager is the interface through which database query operations are provided to Django models. At least one Manager exists for every model in a Django application.

从 Django 文档得知“Manager 是 Django 用来进行数据库查询的一个接口，在 Django 应用中每个 model 都需要至少有一个 Manager”。

# 0X01 Manager 的用法与自定义

我们正常来说用的 `filter/exclude/first/last` 这种查询都是用的 Manager，用法大家是都会用的，不过自定义的话就是另一回事儿了。我们假设有下面这个 model

```python
    class Student(models.Model):
        name = models.CharField()
        age = models.IntegerField()
        gender = models.CharField()
        birthday = models.DateField()
        height = models.DecimalField()
        weight = models.DecimalField()
        remark = models.TextField()

        class Meta:
            verbose_name = '学生'
```

虽然我们没有手动指定 `objects`，但是其实已经从 `models.Model` 继承来了。如果我们非要手动指定的话，可以`objects = models.Manager()`。**值得注意一点是，如果我们手动指定了 Manager 的话，Django 就不会再给我们一个可用的 objects 了。**

下面我们来说一下关于自定义 Manager 的方法，和什么时候需要自定义 Manager。通常来说，没有必须要用 Manager 才能完成的操作，但是很多时候利用自定义 Manager 会帮我们节省很多时间和代码。我们考虑这么一种情况，上面那个 model 定义了一个学生信息表，我们在系统里需要非常经常地使用“身高超过 180，且体重在 70 到 80 之间，且成年的男生；身高超过 170，且体重在 55 到 65 之间，且成年的女生；当天过生日的所有同学”这三种筛选条件。当然，最简单的方法就是每次使用的时候都去 `filter` 一遍，这也没错，不过这样就太奇怪了。稍微有点编程经验的可能会写出如下代码：

```python
    # 这函数名确实太长了，就不乱编了，三个函数对应上面三个查询
    def get_student_1():
        return Student.objects.filter(gender='M', height__gt=180, weight__range=(70, 80), age__gte=18)

    def get_student_2():
        return Student.object.filter(gender='M', height__gt=170, weight__range=(55, 65), age__gte=18)

    def get_student_3():
        today = datetime.date.today()
        return Student.objects.filter(birthday__month=today.month, birthday__day=today.day)
```

调用的时候每次都是 `studnet_1_queryset = get_student_1()`。尤其是如果要在这个基础上增加新的筛选条件，画风就会变成这样：`student_x_queryset = get_student_1().filter(xxxx).exclude(yyyyy)`。就.......也不是不行，不过看起来确实很奇怪，而且跟使用 Manager 的方式比起来也非常不 Pythonic。

如果使用使用 Manager 的方式编写的话就可以是下面这个样子：

```python
    class SpecialStudentManager(models.Manager):
        # 自定义一个方便获取这三种数据的 Manager
        def student_1(self):
            return self.filter(gender='M', height__gt=180, weight__range=(70, 80), age__gte=18)

        def stuent_2(self):
            return self.filter(gender='M', height__gt=170, weight__range=(55, 65), age__gte=18)

        def student_3(self):
            return self.filter(birthday__month=today.month, birthday__day=today.day)


    class Student(models.Model):
        name = models.CharField()
        age = models.IntegerField()
        gender = models.CharField()
        birthday = models.DateField()
        height = models.DecimalField()
        weight = models.DecimalField()
        remark = models.TextField()

        objects = SpecialStudentManager()   # model 这里就改了这一行

        class Meta:
            verbose_name = '学生'
```

> 虽然改动了 Model 但是没有改动数据库表，所以不需要`migrations`。

现在我们尝试调用一下上面定义的几个查询：`Student.objects.student_1()`这样就可以了，而且如果用`@property`装饰上面的方法的话，会更好一些。当然了，我们还可以扩展一下用法：

```python
    class SpecialStudentManager(models.Manager):

        @property
        def student_count(self):    # 所有学生人数
            return self.all().count()

        @property
        def student_name_list(self):    # 所有学生人名列表
            return list(self.all().values_list('name', flat=True))
```

不过可以看出来，正如上面所说“一切用自定义 Manager 实现的功能，都可以不用自定义 Manager 实现”，但是自定义 Manager 的做法也确实让我们的代码更清晰明了，后面改起来也更舒服（不用为了一个常用的筛选去改多处代码），可读性也更高了。
