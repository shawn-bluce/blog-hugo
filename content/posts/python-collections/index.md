---
title: "Python 标准库之 collections"
slug: "python-collections"
date: "2024-01-04T14:12:00+0000"
lastmod: "2025-01-16T09:56:17+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
# 0X00 Header

相信各位肯定都对 Python 中的基础、常见数据类型和数据结构比较熟悉了吧，不管是 `int`、`float`、`string`、`bool` 还是 `list`、`tuple`、`set` 用起来应该也都是手到擒来了吧。下面我们就来简单了解一下相对高级一些的 Python 内置数据结构，这些数据结构全都在 `collections` 的标准库中。

掌握这些数据结构虽然并不能让你「精通 Python」，但起码可以让你的代码更加 Pythonic，也能让你少写几行冗余的代码。

# 0X01 ChainMap

首先是 `ChainMap`，光是看名字大概都能猜到这东西的用途了：把 `Map` 组装为 `Chain` 嘛。在 Python 中最常见的也就是字典了，所以这个数据结构的主要功能就是将多个字典加在一起。

如果你手上有 100 个字典，现在你需要将他们加在一起，在没有 `ChainMap` 的时候大概率会写出这样的代码：先整一个 result 作为最终结果的容器，然后遍历这 100 个字典，一遍遍 update

```python
    dict_list = [dict_0, dict_1, dict_2.......]

    result = dict()
    for d in dict_list:
        result.update(d)

    print(result)
```

现在有了 `ChainMap` 之后可以写成这样：将所有的 `dict` 都作为 `ChainMap` 参数传进取，它就自己帮你拼好了。

```python
    from collections import ChainMap

    dict_list = [dict_0, dict_1, dict_2.......]
    result = ChainMap(*dict_list)
```

> 有一点需要注意的是，使用第一段代码时由于是从前向后遍历 `update` 所以是用后面的值覆盖前面的值；而使用 `ChainMap` 方式则不会覆盖，换句话说就是「先入为主」了
>
> 可以用下面这段代码来测试这个说法

```python
    from collections import ChainMap

    a = {'a': 1, 'b': 2, 'c': 3}
    b = {'a': 111, 'c': 333, 'd': 444}

    res = dict()
    res.update(a)
    res.update(b)
    print(res.get('a'))

    res = ChainMap(a, b)
    print(res.get('a'))
```

# 0X02 Counter

类似这么个需求：有一个列表，里面全是单次，需要统计这些单次出现的次数。简单的写法基本上就是

```python
    word_list = ['hello', 'world', .....]

    count = dict()
    for word in word_list:
        if word in count:
            count[word] += 1
        else:
            count[word] = 1

    print(count)
```

但是这一大坨代码看起来就还是很麻烦，这时候如果有一个现成的 `count` 出现就好了。`Counter` 其实就是来解决这类问题的，我们来看一下用 `Counter` 解决这个问题是怎样的

```python
    from collections import Counter

    word_list = ['a', 'b', 'c', 'c', 'd', 'e', 'a', 'a', 'a', 'a', 'a']

    count = Counter()
    for word in word_list:
        count[word] += 1

    print(count)
```

可见 `Counter` 类其实就是个字典的子类，每次 key 不存在的时候就当它是一个 0。改用了 `Counter` 之后数数的过程一下就从 5 行变为了 2 行。如果说只是节省这三两行代码，那确实没什么必要这么大动干戈。`Counter` 还为我们提供了 5 个非常常用的方法：

  * `elements()` 返回迭代器，每个 key 会出现 value 次，当 value 小于 1 时忽略
  * `most_common(n)` 返回最常见的 n 个元素，也就是 value 最大的 n 个元素
  * `update(c)` 与 `dict` 的普通 `update` 不同，这里是计算加法，也就是 `c1.update(c2)` 意味着 `c1[x] + c2[x]`（相同 key 的 value 做加法）
  * `subtract(c)` 与上面的 `update` 类似，这里是减法
  * `total()` 算总量

这里还是给出这五个方法的例子

```python
    from collections import Counter

    word_list = ['a', 'b',  'b', 'c', 'c', 'd', 'e', 'a', 'a', 'a', 'a', 'a']

    count = Counter()
    for word in word_list:
        count[word] += 1

    print(count)
    # OUTPUT: Counter({'a': 6, 'b': 2, 'c': 2, 'd': 1, 'e': 1})

    print(list(count.elements()))
    # OUTPUT: ['a', 'a', 'a', 'a', 'a', 'a', 'b', 'b', 'c', 'c', 'd', 'e']

    print(count.most_common(3))
    # OUTPUT: [('a', 6), ('b', 2), ('c', 2)]

    print(count.total())
    # OUTPUT: 12


    new_word_list = ['a', 'b', 'c', 'd', 'e']
    new_count = Counter()

    for word in new_word_list:
        new_count[word] += 1

    count.update(new_count)
    print(count)
    # OUTPUT: Counter({'a': 7, 'b': 3, 'c': 3, 'd': 2, 'e': 2})

    count.subtract(new_count)
    print(count)
    # OUTPUT: Counter({'a': 6, 'b': 2, 'c': 2, 'd': 1, 'e': 1})
```

# 0X03 deque

了解的同学肯定了解，栈和队列基本是继链表之后最简单的数据结构了。这个 `deque` 就是在队列的基础上有一个简单变种的版本：双向队列。队列本身规则很简单，就是单纯的先进先出（FIFO），基本操作也就只有两个：入队和出队。这双向队列就是将连个基本操作改成四个了：左右入队和左右出队。

当双向队列的长度没有收到限制时，两侧可以自由的入队出队；当长度受到限制时，队列爆满后继续入队就会导致另一侧强制出队。

Python 中的 `deque` 提供了下面这些方法：

  * `append(x)` 从右侧入队
  * `appendleft(x)` 从左侧入队
  * `clear()` 清空队列
  * `copy()` 创建一份浅拷贝
  * `count(x)` 统计 x 出现了多少次
  * `extent(iterable)` 将可叠戴对象从右侧扩展进来
  * `extentleft(iterable)` 从左侧
  * `index(x)` 返回 x 在队列中的位置，找不到会 ValueError
  * `insert(i, x)` 在队列中指定位置插入，队列爆满会 IndexError
  * `pop()` 从右侧出队
  * `popleft()` 从左侧出队
  * `remove(x)` 找到第一个 x 并删除，找不到会 ValueError
  * `reverse()` 反转队列
  * `rotate(n)` 向右循环 n 位，n 为负数时就是向左。向右循环的意思是：右侧出队一个元素，从左侧入队，也就是说`d.appendleft(d.pop())`

还提供了一个只读属性：`maxlen`，为 None 就是无限大，只有在 `deque(maxlen=n)` 初始化的时候可以指定

# 0X04 defaultdict

顾名思义：这是一个带有默认值的字典。我们可以在实例化它的时候指定默认值是什么，这里用它来搞一个非常简单的 Counter 吧

```python
    from collections import defaultdict

    count = defaultdict(int)
    text = 'aaaaaaaabbbbbbbbbcccccccccc'
    for c in text:
        count[c] += 1

    print(count)
    # OUTPUT: defaultdict(<class 'int'>, {'a': 8, 'b': 9, 'c': 10})
```

这里可以看到我们在实例化的时候给他规定了一个类型 `int`，它的默认值就是 0 了，同样还可以用 `dict`、`list`、`set` 这些类型。

# 0X05 namedtuple

如果我们想存下一台电脑的配置，那么用普通的一堆变量可以、用 `dict` 可以，甚至用 `class` 也可以。但是有这么一个更好用且更合适的东西：`namedtuple`，中文叫命名元组或者具名元组。

```python
    from collections import namedtuple

    Computer = namedtuple('Computer', ['CPU', 'GPU', 'Memory', 'Storage', 'OS'])

    macbook_14 = Computer('M2Max', 'M2Max', '32G', '1T', 'macOS')
    pc = Computer('i7-13900k', 'RTX 4090', '64G', '4T', 'Linux')
    another_pc = Computer(GPU='GT710', CPU='AMD A8', OS='Windows 7', Storage='500G', Memory='8G')

    print(macbook_14)
    print(pc.CPU, pc.GPU, pc.OS)
    print(another_pc)

    # OUTPUT:
    # Computer(CPU='M2Max', GPU='M2Max', Memory='32G', Storage='1T', OS='macOS')
    # i7-13900k RTX 4090 Linux
    # Computer(CPU='AMD A8', GPU='GT710', Memory='8G', Storage='500G', OS='Windows 7')
```

# 0X06 OrderedDict

本来它最大的特性就是会保留插入顺序，但是自从 Python 3.7 的更新使普通的 `dict` 也是有序的之后，这个 `OrderedDict` 就变得略微有那么一点点尴尬了。

不过虽然普通的 `dict` 也开始有序了，但是 `OrderedDict` 也还是多少有一些不同的

  * 常规的 `dict` 擅长映射，排序是次要的
  * `OrderedDict` 排序性能更好，但是空间效率和操作性能是相对更差的
