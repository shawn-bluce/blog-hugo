---
title: "Python 中的迭代、生成和 yield 关键字"
slug: "python-iteration-yield"
date: "2017-11-12T14:40:00+0000"
lastmod: "2025-01-16T10:10:23+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
# 0X00 可迭代对象

Python中的列表，元组，字典，文件都是可迭代对。可迭代对象简单地说就是可以用`for i in xxx:`来遍历的对象。

```python
    my_list = [1, 2, 3, 4, 5, 6, 7]
    for i in my_llist:
        print i

    my_dict = {
        'a': u'苟利国家生死以',
        'b': u'岂因祸福避趋之'
    }
    for i in my_dict:
        print i
```

不过如果数据量非常非常庞大的时候，会很影响程序的性能。这种时候就可以使用生成器来解决这个问题。

# 0X01 生成器

生成器的用法和普通的可迭代对象差不多，最大的特点就是：“用的时候才去计算”。这里写一个简单的例子，演示一下情况。第一种生成超大列表的方式要逐项计算完才算弄出来了这个10000长的列表，而后者是生成器，只是声明了怎么算，并没有真的去算，所以在速度上才是完全不能比的。

```python
    #!/usr/bin/env python
    # coding=utf-8

    import time

    if __name__ == '__main__':
        start = time.time()
        [x**x for x in range(10000)]    # 这里生成的是一个超大的列表
        end = time.time()
        print end - start

        start = time.time()
        (x**x for x in range(10000))    # 这里生成一个超大的生成器（注意看，括号不一样）
        end = time.time()
        print end - start
```

运行结果如下：

```
    shawn in ~ λ python hello.py
    6.76475715637
    0.000135898590088
```

# 0X02 yield关键字

那是时候自己生成弄一个生成器出来了，Python中提供的yield关键字就是用来干这个的，通过这个关键字可以创造自己的生成器。
还是上面同样的例子，稍加改动

```python
    #!/usr/bin/env python
    # coding=utf-8

    import time

    def create_list():
        # 创建一个普通的列表
        my_list = []
        for x in range(10000):
            my_list.append(x**x)
        return my_list

    def test_yield():
        # 使用yield
        for x in range(10000):
            yield x**x

    if __name__ == '__main__':
        start = time.time()
        a = create_list()
        end = time.time()
        print end - start

        start = time.time()
        a = test_yield()
        end = time.time()
        print end - start
```

运行起来看到的时间差距和刚刚演示的也差不多

```
    shawn in ~ λ python hello.py
    6.45007514954
    0.0080668926239
```