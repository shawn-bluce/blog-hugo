---
title: "Python 中的 filter 与 map/reduce 方法"
slug: "python-filter-map-reduce"
date: "2020-05-25T14:42:00+0000"
lastmod: "2025-01-16T10:06:26+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
# 0X00 前言 & Pythonic

Python管`filter/map/reduce`这些叫高阶函数，听起来有点高级有点难搞的意思，实际上是贼简单的东西。下面通过几个简单的例子来帮助大家了解一下`filter/map/reduce`这三个高阶函数的简单用法。

事先声明，这三个函数都是扩展性质的东西，从来不用这三个函数也可以正常的编写程序，没有什么功能是没了这三个函数就写不出来的。只不过是这三个函数的出现能让之前很丑陋的代码变得精简易读了而已。

这三个函数非常适合搭配`lambda`来使用，编写非常Pythonic的代码，具体什么是Pythonic其实很难定义，其实就是把Python编程一个形容词了，比如你看到一个人“穿了运动鞋牛仔裤帽衫双肩包黑框眼镜电子表”就会说他“太程序员了”，大概就是这么个意思。总结来说呢就是 **非常具有Python特色的Python代码** 。比如下面这段代码明显就不Pythonic

```python
    for index in range(len(name_list)):
        print(name_list[index])
```

而这种代码就是Pythonic的写法

```python
    for name in name_list:
        print(name)
```

尤其是结合了`lambda`之后，就能写出更Pythonic的代码了，例如

```python
    def is_boy(student):
        if student.gender == 'M':
            return True
        else:
            return False
```

就可以直接用`lambda`改写成这个样子

```python
    is_boy = lambda studnet: student.gender == 'M'
```

# 0X01 filter

`filter`顾名思义，一定是一个筛选器。当我们有一个列表想要找出这个列表中满足某些条件的数据，在不是用`filter`的情况下很有可能会写出这样的代码

```python
    number_list = [1, 2, 3, 4, 5, 6, 7, 8, 9]

    def is_odd(input_number):
        if input_number == 0:
            raise Exception('不考虑0的问题，我们现在是研究filter')
        return input_number % 2

    odd_list = []
    for number in number_list:
        if is_odd(number):	# 把奇数怼到新列表里
            odd_list.append(number)

    print(odd_list)
```


但是借助`filter`就可以将新列表的生成变得很简单，本质上只要了一行代码。

```python
    odd_list = filter(is_odd, number_list)

    print(list(odd_list))
```

`filter`接受两个参数，第一个参数是个只接受一个参数的`function`，第二个参数是个列表。工作原理就是将列表中的对象一个个塞到第一个参数的`function`中取得返回值，将返回值为`True`的保存到新列表中，最终返回。

> Python2中`filter`返回的直接是列表，而Python3中返回的则是可迭代对象。

# 0X02 map

`map`可以用来批量处理数据，批量传参。`map`接受不定长度的参数，其中第一个参数固定为一个“接受n个参数”的function，然后后面紧跟真就是n个可迭代的参数。

```python
    #!/usr/bin/env python3
    # coding=utf-8


    # 接受n个参数
    def join_name(first_name, middle_name, last_name):
        """拼接三个字符串"""
        return '{}_{}_{}'.format(first_name, middle_name, last_name)


    if __name__ == '__main__':
        first_name_list = ['zhang', 'li', 'wang', 'zhao']
        middle_name_list = ['a', 'b', 'c', 'd']
        last_name_list = ['san', 'si', 'wu', 'liu']

        # 方法1，使用map传递n+1个参数，第一个是function对象，后面的是参数列表
        for result in map(join_name, first_name_list, middle_name_list, last_name_list):
            print(result)


        # 方法2，不使用map，相当于下面这种
        for index in range(len(first_name_list)):
            result = join_name(first_name_list[index], middle_name_list[index], last_name_list[index])
            print(result)
```

输出的内容都是这个样子的：

![](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/20200525234648.png)

使用了`map`之后不仅在for那里不用非常蠢得用`range(len(first_name_list))`了，也不用拿到下标之后到处跑着用下标取值了。而且有一个很大的优点是：`map`自动使用多个参数列表中最短的那个，也就是说不会出现`IndexError: list index out of range`的数组越界问题了。

# 0X03 reduce

`reduce`方法从Python 3开始就不是全局命名空间里的function了（说人话就是挪到官方包里去了，需要导入，不能直接用了），所以我们需要`from functools import reduce`才能用到。`reduce`相对更简单一些，固定接受两个参数，第一个参数是一个“固定接受两个参数的”function，第二个参数是一个可迭代对象。具体用法可以看示例，非常通俗易懂。

```python
    #!/usr/bin/env python3
    # coding=utf-8

    from functools import reduce


    def join_chars(char_a, char_b):
        """拼接两个字符串, reduce的第一个参数只能是一个接受两个参数的function"""
        return char_a + char_b


    if __name__ == '__main__':
        char_list = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n']
        print(reduce(join_chars, char_list))	# 使用reduce

        # 其实这个reduce就类似于下面这种写法
        res = join_chars(char_list[0], char_list[1])
        res = join_chars(res, char_list[2])
        res = join_chars(res, char_list[3])
        res = join_chars(res, char_list[4])
        res = join_chars(res, char_list[5])
        res = join_chars(res, char_list[6])

        # 或者类似这种
        res = join_chars(
            join_chars(
                join_chars(
                    join_chars(
                        char_list[0],
                        char_list[1]
                    ),
                    char_list[2]
                ),
                char_list[3]
            ),
            char_list[4]
        )
```

# 0X04 结合lambda

结合`lambda`之后可以将上面的`is_odd`改写成

```python
    #!/usr/bin/env python3
    # coding=utf-8

    number_list = [1, 2, 3, 4, 5, 6, 7, 8, 9]

    odd_list = filter(lambda number: number % 2, number_list)

    print(list(odd_list))
```

可以将上面`map`中的代码改写成这样：

```python
    #!/usr/bin/env python3
    # coding=utf-8

    if __name__ == '__main__':
        first_name_list = ['zhang', 'li', 'wang', 'zhao']
        middle_name_list = ['a', 'b', 'c', 'd']
        last_name_list = ['san', 'si', 'wu', 'liu']

        # 方法1，使用map传递n+1个参数，第一个是function对象，后面的是参数列表
        for result in map(
            lambda a, b, c: '{}_{}_{}'.format(a, b, c), first_name_list, middle_name_list, last_name_list
        ):
            print(result)
```


可以将上面`reduce`中的代码改成这样：

```python
    #!/usr/bin/env python3
    # coding=utf-8

    from functools import reduce

    if __name__ == '__main__':
        char_list = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n']
        print(reduce(lambda a, b: a + b, char_list))
```