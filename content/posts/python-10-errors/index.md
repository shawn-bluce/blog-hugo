---
title: "编写 Python 程序的 10 个典型错误"
slug: "python-10-errors"
date: "2025-06-20T15:45:27+0000"
lastmod: "2025-06-20T15:46:06+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
# 0X00 开头

最近买了本书，叫做《100个Go语言典型错误》，发现这样的总结很有意思。决定自己也写一个，不过以我的水平写本书还是有点离谱了，但是写一篇博客还是没什么问题的，所以就有了这篇文章。

# 0X01 函数默认值传递空列表

这是一个典型错误，很多很多的 Python 程序员都犯过这个错误。一般在定义一个函数且给它设置默认值的时候我们都会写成 `def foo(a=0, b="", c=None)` 这个样子，这种写法是完全没有问题的。但是有时候也会写成：`def foo(d=[])`，这就完犊子了～

我们看下面这段代码

```python
    def foo(a=[]):
        a.append('x')
        print(a)

    for i in range(10):
        foo()
```

你以为会输出 10 个 `['x']`？那就大错特错了，真正的输出是这样的：

```
    ['x']
    ['x', 'x']
    ['x', 'x', 'x']
    ['x', 'x', 'x', 'x']
    ['x', 'x', 'x', 'x', 'x']
    ['x', 'x', 'x', 'x', 'x', 'x']
    ['x', 'x', 'x', 'x', 'x', 'x', 'x']
    ['x', 'x', 'x', 'x', 'x', 'x', 'x', 'x']
    ['x', 'x', 'x', 'x', 'x', 'x', 'x', 'x', 'x']
    ['x', 'x', 'x', 'x', 'x', 'x', 'x', 'x', 'x', 'x']
```

为什么呢？因为函数在定义的时候就将空列表初始化了，并且记录下了引用，以后每次都会对同一个列表进行操作。另外不只是列表，字典作为默认值也会有这个问题的。

# 0X02 对闭包的无意识理解

很多人对闭包的理解就是粗浅的「函数里定义函数」，当然这是没错的，但是缺少了一些东西。如果只是函数里定义函数的话，下面这段代码会发生什么？

```python
    def foo():
        x = 2
        def bar():
            print(x)
        return bar

    foo()()
```

没错，这段代码是会正确输出 `2` 的，因为闭包还有一个特性就是内部函数能访问外部函数中的变量，即使外部函数已经运行完毕了。

# 0X03 使用不合适的变量名

这一点虽然很简单，但确实很容易出现，比如用 `list` / `dict` / `str` 做变量名。因为 Python 中这 3 个名字是非常非常常用的`type/function`，所以万万不可用。

# 0X04 字符串拼接的性能问题

少量字符串拼接使用 `+` 当然没有任何问题，但是如果量很大的话就不建议使用 `+` 了，使用 `join` 会快非常多。

```python
    import time

    COUNT = 10000000

    # 使用 + 的版本
    start_time = time.time()
    result1 = ''
    for i in range(COUNT):
        result1 += 'x'
    end_time = time.time()
    plus_time = end_time - start_time

    # 使用 join 的版本
    start_time = time.time()
    char_list = []
    for i in range(COUNT):
        char_list.append('x')
    result2 = ''.join(char_list)
    end_time = time.time()
    join_time = end_time - start_time

    print(f"+ 方法耗时: {plus_time:.4f}秒")
    print(f"join 方法耗时: {join_time:.4f}秒")
```

在我的电脑上运行是这样的：

```
    + 方法耗时: 1.0472秒
    join 方法耗时: 0.5272秒
```

# 0X05 错误使用字符串的 strip 方法

学过 Python 的肯定都用过字符串的 `strip` 方法或者它衍生出来的 `lstrip` 和 `rstrip`，比如我们会用 `'hello'.rstrip('o')`来去掉字符串右侧的 `o`。

也有些人会用下面这种方法去掉额外的字符：

```python
    phone_num = "+8613588888888"
    phone_num = phone_num.strip("+86")
```

因为 `strip('o')` 可以去掉 `o`，就自然而然以为 `strip('+86')` 就是去掉 `+86` 了。但你实际运行起来就会发现 `phone_num` 就只剩下 `135` 了。因为 `strip('+86')`的意思并不是去掉两侧的 `+86` 而是去掉两侧的 `+`/`8`/`6`，凑巧这个手机号后面全是 `8` 所以就全没了 🤷‍♂️

# 0X06 认为 Python 的多线程没有用

这是一个典型谣言了，很多人一旦听说 Python 有 GIL 之后就大张旗鼓的说：“Python 的多线程屁用没有”。但实际上真的是这样吗？并不是。

Python 的多线程如果用在 CPU 密集型的计算任务上，那确实没什么用；但是如果用在 IO 密集型的任务上，那和真正意义上的多线程是没有显著差别的。所以严谨来说 Python 确实没有真正意义上的多线程，但也不能说 Python 的多线程没有用。

# 0X07 过分的一行流

有些 Python 程序员推崇精简代码，使用各种列表生成式、字典生成式之类的，这当然是没问题，精简又好看，而且这是非常 Pythonic 的写法。但是有些人有些过分，强行把多行代码挤成一行，或者强行使用 Python 的高级语法。

比如说 `a = {v: k for k, v in my_dic.items()}` 这种代码，其实挺好的，但是有些人会写这种东西：

```python
    result = {f"item_{i}": [x*2 for x in [y+1 for y in range(3)] if x in [z for z in range(10) if z % 2 == 0]] for i in range(3)}
```

强吗？强，对语言没点理解是写不出来能跑的这种代码的。但是好吗？codereview 的时候可能会被同时打死。

# 0X08 手撸 csv 文件

是的没错，都 2025 年了还有人在手撸 csv 文件。赶紧了解一下 `csv` 库的用法吧，不要再去折磨那个逗号和转义符了。

写入：

```python
    import csv

    # 学生信息数据
    students = [
        ['姓名', '年龄', '班级', '成绩'],
        ['张三', 18, '高一(1)班', 85],
        ['李四', 17, '高一(2)班', 92],
        ['王五', 18, '高一(1)班', 78],
        ['赵六', 17, '高一(3)班', 88]
    ]

    # 写入CSV文件
    with open('students.csv', 'w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerows(students)

    print("学生信息已写入 students.csv 文件")
```

读取：

```python
    import csv

    # 读取CSV文件
    with open('students.csv', 'r', encoding='utf-8') as file:
        reader = csv.reader(file)

        print("学生信息列表：")
        for row in reader:
            print(f"{row[0]:<8} {row[1]:<6} {row[2]:<12} {row[3]}")

    # 也可以使用字典方式读取
    print("\n使用字典方式读取：")
    with open('students.csv', 'r', encoding='utf-8') as file:
        reader = csv.DictReader(file)

        for row in reader:
            print(f"姓名: {row['姓名']}, 年龄: {row['年龄']}, 班级: {row['班级']}, 成绩: {row['成绩']}")
```

# 0X09 文件硬读写

我们有时候会用 `open('xxx', 'r').read()` 直接把一个文件读到内存里，如果文件比较小确实是可以这样干的，读到内存里直接当字符串处理就好。但是如果文件很大，就不要一直这么搞了，你 1G 的日志文件还通过这种方式读取，服务都要被卡死了。

```python
    with open('xxx.log', 'r') as f:
        for line in f:
            pass
```

这样我们的 f 就是一个类似生成器的东西了，每次只读一行，就不会再被这么初级的 IO 问题限制性能了。

# 0X0A 认为 Python 字典是无序的

是的兄弟，2025年了，Python 的字典早就不是无序的了，甚至是 7 年前的 Python 3.7 发布的时候 dict 就不是无序的了。

[Python 官方文档](<https://docs.python.org/3/library/stdtypes.html#dict>)贴在这里：

> Changed in version 3.7: Dictionary order is guaranteed to be insertion order. This behavior was an implementation detail of CPython from 3.6.
