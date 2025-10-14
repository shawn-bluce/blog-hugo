---
title: "Python 中格式化字符串的几种方式"
slug: "python-string-format"
date: "2021-10-12T12:52:00+0000"
lastmod: "2025-01-17T01:48:10+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
# 0X00 前言

如果秉承着「能用就行」的原则，那么这篇文章提到的东西基本都没什么卵用；如果秉承着「写更好的代码」的原则，那么这里提到的东西也许对你有所帮助。

> 内容主要取材自 _Effective Python_ ，主要是作为自己学习后的一个输出而总结的这篇博客

# 0X01 使用 % 的 C 风格格式化

首先是沿用自 C 风格的使用 % 进行的字符串格式化方法：

```python
    >>> name = 'Shawn'
    >>> job = 'developer'
    >>> text = 'I am %s, a %s' % (name, job)
    >>> text
    'I am Shawn, a developer'
    >>> height = 123.456
    >>> text = 'My height is %4d.' % height
    >>> text
    'My height is  123.'
    >>> text = 'My height is %4f.' % height
    >>> text
    'My height is 123.456000.'
    >>> text = 'My height is %4.8f.' % height
    >>> text
    'My height is 123.45600000.'
```

这种写法在写惯了 C 的人身上比较常见，比较熟悉而且也比较简单。不过这种写法有几个问题，首先就是**当百分号右侧的变量数量发生变化或者类型发生变化的时候，程序很有可能因为类型转化出现不兼容的情况** （当然了，本来是 `%s %4d` 对应字符串和数字，现在两个都是字符串了当然就出错了）。如果要解决这种问题的话，必须每次修改都要检查百分号左右的占位符和具体数值是否能对应的伤，而且一旦占位符多了之后还很容易看花眼。

还有一个问题就是**填充数值的时候通常需要对具体的值进行一些处理，比如保留某几位长度之类的，这样一来表达式可能会很长，从而显得很混乱** 。

```python
    coffee_price_list = [
        ('Americano', 15),
        ('Latte', 25),
        ('Cappuccino', 30)
    ]

    for index, (name, price) in enumerate(coffee_price_list):
        content = '%d.  %-12s  -----  %.2f' % (index, name, price)
        print(content)

    # output
    0.  Americano     -----  15.00
    1.  Latte         -----  25.00
    2.  Cappuccino    -----  30.00
```

我们来看 `for` 循环里面那行，是不是确实看起来乱乱糟糟的，这还只是三个占位符，如果更多的话就会更混乱了。

第三个问题是**如果要用同一个值来填充多个位置，那就需要在右侧重复多次** （废话之：你想要几个就得写几个）。我们假设你有一个保证书模板，只需要填入姓名、错误和保证内容就可以生成出例如「我XX再也不YY了，我保证以后ZZ」的十万字长文。但是整篇文章里出现了大量的空位，需要填入这些 XX/YY/ZZ 怎么搞呢？你可能需要在后面写上不计其数的 `'--------%s-------%s-------%s-------%s' % (xx, xx, yy, zz, zz, yy)` 这种东西（别跟我说你要用 `replace` 那是另外的内容，这里只讨论字符串格式化😅）。

当然了这个问题也不是无解，我们使用 `dict` 来替换平时用的 `tuple` 就可以了，就是类似下面这种用法（虽然我从来没真的在代码里见过谁这么写）

```python
    value_a = 'aaaaaaaaaaaaaaaa'
    value_b = 'bbbbbbbbbbbbbbbb'
    value_c = 'cccccccccccccccc'

    content = '''
        %(val_a)s, %(val_a)s, %(val_a)s
        %(val_b)s, %(val_b)s, %(val_b)s
        %(val_c)s, %(val_c)s, %(val_c)s
        %(val_a)s, %(val_b)s, %(val_c)s
        ''' % {
            'val_a': value_a,
            'val_b': value_b,
            'val_c': value_c
        }

    print(content)

    # output

        aaaaaaaaaaaaaaaa, aaaaaaaaaaaaaaaa, aaaaaaaaaaaaaaaa
        bbbbbbbbbbbbbbbb, bbbbbbbbbbbbbbbb, bbbbbbbbbbbbbbbb
        cccccccccccccccc, cccccccccccccccc, cccccccccccccccc
        aaaaaaaaaaaaaaaa, bbbbbbbbbbbbbbbb, cccccccccccccccc
```


虽然这种写法解决了多次重复使用的问题，但是加重了第二点也就是**代码更冗长** 了，因为不仅要给变量做格式化，还要给每个占位符再设定一个 `key` 且为其匹配好。

最后就是因为每次都需要把 `key` 至少写两次（占位符那里一次，后面的字典里一次），甚至因为 `value` 过长还可能再把变量提出去单独定义一下，就会导致整个表达式非常长，比较容易出现 bug 且定位 bug 比较复杂。

# 0X02 另一种方法：format函数与方法

`format` 是我平时用的最多的一种方法了，比较常规的方法是调用`str` 对象的 `format()` 方法，例如下面这样

```python
    name_1 = 'shawn'
    name_2 = 'bluce'

    print('hello {}, hello {}.'.format(name_1, name_2))
    print('hello {:<10}, hello {:<20}.'.format(name_1, name_2))
    print('hello {1}, hello {0}.'.format(name_1, name_2))
    print('hello {1}, hello {0}, hello {0}, hello {1}.'.format(name_1, name_2))

    # output
    hello shawn, hello bluce.
    hello shawn     , hello bluce               .
    hello bluce, hello shawn.
    hello bluce, hello shawn, hello shawn, hello bluce.
```

第二种可能比较少见，不过规则比较简单，就是在花括号里写一个冒号，冒号右边可以用 C 方法格式化变量。第三四种就比较常见了，可以通过 `index` 来规定位置。

这种方法还有一些更高级的用法，例如在花括号里访问字典的 `key` 或者访问列表中的下标（好像也没见人这么用过）

```python
    data = [
        {'name': 'shawn', 'gender': 'M'},
        {'name': 'bluce', 'gender': 'F'}
    ]

    print('{data[0][name]} and {data[1][name]} is {data[0][gender]} and {data[1][gender]}'.format(data=data))

    # output
    shawn and bluce is M and F
```

# 0X03 更好的方法 f-string 插值格式字符串

在 Python 3.6 中引入的这个特性可以解决上述提到的问题，语法要求格式化的字符串前面加上一个`f`做前缀，就类似于之前的`b/r`这种。这里也同样支持前面 `format` 那里用到的格式化方法，例如 `f'{name_1:<10}, {value:.2f}'` 这种。

一个简单的例子

```python
    name_1 = 'shawn'
    name_2 = 'bluce'

    print(f'hello {name_1}, hello {name_2}')

    # output
    hello shawn, hello bluce
```

我们现在再回过头来看一下最开始提到的四个问题：**第一个** ，如果需要调整顺序，那么百分号左侧的正文要改，右侧的值也要改，就要改两次。现在没有百分号也就不再区分左右了，如果调整顺序那么就只调整一次就行，方便了很多。**第二个** ，如果对填进去的值稍作处理可能会导致整个表达式变得很长。现在因为省略了百分号右边的内容，所以整个表达式还是精简了不少的。**第三个** ，当某个变量/值要用多次的时候就需要左右共写两次那么多。用 `f-string` 方式的话，如果确实需要调用多次且每次都要进行修改（例如保留小数或是转成大写之类的），则可以考虑将其提取出去单独赋值，然后在格式化的时候用新值来代替，还能更加符合字符串格式化的语义。**第四个** 是说如果使用 `dict` 的话会使代码变多，现在不用字典了当然也就没有这个问题了。

下面是几种用法，看起来 `f-string` 并没有代码量少很多，是因为这个例子并不能很明显的体现出代码量少的优势，但是已经体现出可读性和维护性的优势了。如果一眼看过去，明显是 `f-string` 的用法最简单清晰明了。

```python
    data = [
        {'name': 'shawn', 'score': 78.5},
        {'name': 'ami', 'score': 89.0},
        {'name': 'jack', 'score': 92.0},
        {'name': 'amber', 'score': 99.5}
    ]

    print('------------- style_1 -------------')
    for item in data:
        style_1 = 'name: %-10s score: %2.2f' % (item['name'], item['score'])
        print(style_1)

    print('\n\n------------- style_2 -------------')
    for item in data:
        style_2 = 'name: {:10s} score: {:2.2f}'.format(item['name'], item['score'])
        print(style_2)

    print('\n\n------------- style_3 -------------')
    for item in data:
        style_3 = 'name: {name:10s} score: {score:2.2f}'.format(name=item['name'], score=item['score'])
        print(style_3)

    print('\n\n------------- f-string -------------')
    for item in data:
        f_string = f'name: {item["name"]:10s} score: {item["score"]:2.2f}'
        print(f_string)

    # output
    ------------- style_1 -------------
    name: shawn      score: 78.50
    name: ami        score: 89.00
    name: jack       score: 92.00
    name: amber      score: 99.50


    ------------- style_2 -------------
    name: shawn      score: 78.50
    name: ami        score: 89.00
    name: jack       score: 92.00
    name: amber      score: 99.50


    ------------- style_3 -------------
    name: shawn      score: 78.50
    name: ami        score: 89.00
    name: jack       score: 92.00
    name: amber      score: 99.50


    ------------- f-string -------------
    name: shawn      score: 78.50
    name: ami        score: 89.00
    name: jack       score: 92.00
    name: amber      score: 99.50
```