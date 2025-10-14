---
title: "Python 中的 enumerate() 方法"
slug: "python-enumerate"
date: "2017-12-24T04:00:00+0000"
lastmod: "2025-01-16T10:03:22+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
# 0X00 enumerate是什么

`enumerate()`是一个Python自带的函数，用来同时遍历刻碟带对象和索引值．

# 0X01 enumerate怎么用

如果不在不使用`enumerate()`的情况下去除一个字符串列表中的字符串中的空格，那么通常会写出下面这种程序．

```python
    #!/usr/bin/env python
    # -*- coding: utf-8 -*-

    my_list = ["   搞个大  新闻", "我作  为一 个长者", "比 谁跑 的都快",
               "哪 个国家 我没 去过", "比你们不  知道     高到 哪里去了",
               "谈 笑风生  ", "当然 支持    啊", "遵循  基本法  的   "]
    index = 0
    for item in my_list:
        my_list[index] = item.replace(' ', '')
        index = index + 1
```

可以看到光是处理空格都用了四行，而且还并不怎么优雅．那么可以使用`enumerate()`来修改一下这个程序．

```python
    #!/usr/bin/env python
    # -*- coding: utf-8 -*-

    my_list = ["   搞个大  新闻", "我作  为一 个长者", "比 谁跑 的都快",
               "哪 个国家 我没 去过", "比你们不  知道     高到 哪里去了",
               "谈 笑风生  ", "当然 支持    啊", "遵循  基本法  的   "]

    for index, item in enumerate(my_list):
        my_list[index] = item.replace(' ', '')
```

语法大概就是这`for 索引, 对象 in enumerate(可迭代对象)`．用起来不仅干净优雅而且可读性也更强了．
