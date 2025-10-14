---
title: "Python之面向对象 2"
slug: "python-oop-2"
date: "2019-11-19T14:31:00+0000"
lastmod: "2025-01-17T01:43:35+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
# 0X00 Python3的super

Python中对象的概念都快被大家淡忘了，因为一切都是对象（话虽然这么说，但是怎么可能淡忘对象呢）。看下面一段Python2的代码，Python2中麻烦的就是这个`super()`的用法。

```python
    class Human:
        def __init__(self):
            self.name = 'human'
            print 'hello, i im', self.name


    class Student(Human):
        def __init__(self):
            super(Student, self).__init__()
            self.name = 'student'
            print 'hello, i im', self.name


    a = Student()
```

在初学Python的时候，如果是Python2很大可能会在`super(Student, self).__init__()`这段迷惑好一阵子，不过好在[Python2马上就要凉透了](<https://pythonclock.org/>)，在Python3中可以将代码改写成如下方式

```python
    class Human:    # 不用强行继承自object了
        def __init__(self):
            self.name = 'human'
            print('hello, i im', self.name)


    class Student(Human):
        def __init__(self):
            super().__init__()  # super的用法也更明了
            self.name = 'student'
            print('hello, i im', self.name)


    a = Student()
```

其中`super`的用法由`super(Student, self).__init__()`改成了`super().__init__()`，看起来清晰多了，在使用Python3后不建议以任何理由使用老式Python中的`super`调用。

# 0X01 **str**

写一个自己的类通常都需要实现一个`__str__`方法，这个方法用于粗略的展示对象，可以看下面这个例子。

```python
    class Student:
        def __init__(self, name, age, gender):
            self.name = name
            self.age = age
            self.gender = gender

            # 下面这一堆xxx表示其他很多属性
            # self.xxxx = xxxx
            # self.xxxx = xxxx
            # self.xxxx = xxxx
            # self.xxxx = xxxx
            # self.xxxx = xxxx
            # self.xxxx = xxxx
            # self.xxxx = xxxx

        def __str__(self):
            return 'name:{} age:{} gender:{}'.format(self.name, self.age, self.gender)


    a = Student('shawn', '24', 'm')
    b = Student('lucy', '24', 'f')
    c = Student('bill', '24', 'm')

    print(a)
    print(b)
    print(c)
```

可以尝试先把`__str__`的定义注释掉执行一下，看到的输出应该是类似这样的：

```python
    <__main__.Student object at 0x7f875f881d10>
    <__main__.Student object at 0x7f875f881d90>
    <__main__.Student object at 0x7f875f881e10>
```

如果再取消`__str__`的注释，看到的输出就是这样的了：

```
    name:shawn age:24 gender:m
    name:lucy age:24 gender:f
    name:bill age:24 gender:m
```

可以看到输出变成肉眼可识别的了。

> 通过 str(object) 以及内置函数 format() 和 print() 调用以生成一个对象的“非正式”或格式良好的字符串表示。返回值必须为一个 字符串 对象。
> 此方法与 object.**repr**() 的不同点在于 **str**() 并不预期返回一个有效的 Python 表达式：可以使用更方便或更准确的描述信息。
> 内置类型 object 所定义的默认实现会调用 object.**repr**()。 [官方文档](<https://docs.python.org/zh-cn/3/reference/datamodel.html?highlight=repr#object.__str__>)

# 0X02 **repr**

`__str__`其实很多人都是知道的，毕竟这也算是Python中最基础的部分之一了，不过这里的`__repr__`貌似就有些同学不太清楚了。`__repr__`的功能和`__str__`是类似的，不过`__str__`输出的结果是方便肉眼识别的，而`__repr__`输出的结果是”可以通过输出反向还原对象“的，换句话说就是带有对象的详尽信息。

```python
    a = {'a': 1, 'b': 2, 'c': 3}
    print(repr(a))

    b = {'a': a, 'b': a, 'c': a}
    print(repr(a))
```

执行上面这坨代码就理解这个方法的基本情况了，注：`repr(a)`算是`a.__repr__()`的语法糖了，效果相同。

> 由 repr() 内置函数调用以输出一个对象的“官方”字符串表示。如果可能，这应类似一个有效的 Python 表达式，能被用来重建具有相同取值的对象（只要有适当的环境）。如果这不可能，则应返回形式如 <...some useful description...> 的字符串。返回值必须是一个字符串对象。如果一个类定义了 **repr**() 但未定义 **str**()，则在需要该类的实例的“非正式”字符串表示时也会使用 **repr**()。
> 此方法通常被用于调试，因此确保其表示的内容包含丰富信息且无歧义是很重要的。 [官方文档](<https://docs.python.org/zh-cn/3/reference/datamodel.html?highlight=repr#object.__repr__>)
