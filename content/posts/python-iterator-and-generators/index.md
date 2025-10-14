---
title: "Python 中的可迭代对象、迭代器与生成器"
slug: "python-iterator-and-generators"
date: "2019-08-20T14:11:00+0000"
lastmod: "2025-01-16T09:07:05+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
# 0X00 什么是可迭代对象

我们平时用到的`list/set/tuples`是最常见的可迭代对象，简单判断就是说当可以`for item in this_obj`的时候`this_obj`就是可迭代对象。所以不只是`list/set/tuples`，打开的文件或是Django中的`queryset`也都是可迭代对象。

```python
    In [1]: user_list = iter(['shawn', 'danny', 'jenny', 'liming'])

    In [2]: next(user_list)
    Out[2]: 'shawn'

    In [3]: next(user_list)
    Out[3]: 'danny'

    In [4]: next(user_list)
    Out[4]: 'jenny'

    In [5]: next(user_list)
    Out[5]: 'liming'
```

使用`iter()`可以将列表变为一个迭代器，然后使用`next`方法访问下一个元素。实际上`iter(this_obj)`方法和`next(this_obj)`方法分别调用的是`this_obj.__iter__()`和`this_obj.__next__()`，所以我们如果想要自己实现迭代器的话就需要实现这两个方法。

# 0X01 迭代器

那我们来实现实现一个计算斐波纳契数列的迭代器吧。

```python
    class Fib:
        def __init__(self):
            self.pre = 0
            self.cur = 1

        def __iter__(self):
            """迭代器的iter方法返回自身"""
            return self

        def __next__(self):
            """每次计算下一个值"""
            res = self.cur
            self.cur = self.cur + self.pre
            self.pre = res
            return res
```

运行起来：

```python
    >>> fib = Fib()
    >>> for i in fib:
            print(i)
    1
    1
    2
    3
    5
    8
    13
    21
    34
    55
    89
    144
    233
```

这里没有中断的判定，如果需要中断的话就抛出一个`StopIteration`异常就好了，就像这样：

```python
    class Fib:
        .......
        def __next__(self):
            res = self.cur
            self.cur = self.cur + self.pre
            self.pre = res
            if res > 100:   # 数列最后值小于100
                raise StopIteration
            return res
```

# 0X02 生成器

**生成器是一类特殊的迭代器** ，将我们平时写的这种`[i for i in range(100)]`列表生成式的方括号换成小括号，就是将结果从列表变成生成器了`(i for i in range(100))`。

我们可以从这里看出来

```python
    In [1]: a = [i for i in range(100)]

    In [2]: b = (i for i in range(100))

    In [3]: type(a)
    Out[3]: list

    In [4]: type(b)
    Out[4]: generator
```

# 0X03 区别在哪儿呢

既然迭代器用起`for .. in ..`来跟列表之类的差不多，那为什么还要搞这个东西出来呢。显然不是用来玩票的，用处是大大的有。下面来举个例子，有一个10GB大小的日志文件，但是我们机器只有8G的内存，还打算逐行分析日志内容，那肯定就不能直接把文件全部都读入内存了，显然内存是不够用的。这种类似的时候就可以使用迭代器的方法打开这个文件，其实也就是我们常用的这种`for line open('nginx.log')`的方式。这种方式可以发现不管多大的文件，只要不是单行超长的文件，读取起来都是差不多的。

因为迭代器只是每次要用下一行的时候才去读取下一行，而不是把整个文件都读到内存里再在需要的时候去取。简单地来说就是**列表元组之类的结构是保存数据，而迭代器是保存算法** 。
