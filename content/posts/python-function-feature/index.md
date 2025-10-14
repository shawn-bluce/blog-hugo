---
title: "Python 中函数的特性"
slug: "python-function-feature"
date: "2021-05-26T13:39:00+0000"
lastmod: "2025-01-16T10:07:32+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
# 0X00 前言

在正式开始之前我们先要搞明白一个事情，那就是「函数」和「方法」到底有什么区别。首先来看一下在 [Python官方文档](<https://docs.python.org/3/glossary.html>)里的定义。

函数：可以接受零个或几个参数并向调用者返回一些值的一系列语句。

> **function** : A series of statements which returns some value to a caller. It can also be passed zero or more arguments which may be used in the execution of the body.

方法：在类里定义的函数。

> **method** : A function which is defined inside a class body. If called as an attribute of an instance of that class, the method will get the instance object as its first argument (which is usually called self).

但是一般大家并不会很认真的区分「函数」与「方法」，而且就算不区分也并不会对平时的交流甚至编码造成任何影响（起码我没有因为不认真区分它们导致交流出现分歧或者代码出现 bug 的时候）。所以这里列出来也只是提个醒，防止有人并不是很清楚这两个名词表示的含义。其实我就是因为要写这篇博客才去搜了一下它们到底有什么区别，以前都是管 Python 里的叫「方法」，管 C 里的叫「函数」，不知道有没有人也是有这种不良习惯的🤣

既然搞清楚了，那么这篇文章后面就用「函数」来称呼好了，因为这些特性与是否定义在类里没有任何关系。（估计这是我第一次也是最后一次认认真真区分这两个东西了 hhhhh）

# 0X01 万物皆对象

都说 Python 中万物皆对象，那当然函数也是对象，我们可以用下面这段代码来验证一下

```python
    #!/usr/bin/env python3

    def hello():
        print('hello, world')

    print('id is ', id(hello))
    print('type is ', type(hello))

    if isinstance(hello, object):
        print('石锤，就是对象')
```

运行结果可以看到定义好的函数有自己的 id，`type()` 的输出结果也证明了它对象的身份，最后的 `isinstance()` 就更是石锤了。

```python
    id is  140617210113904
    type is  <class 'function'>
    石锤，就是对象
```

# 0X02 函数的本质

那么既然函数也是对象，那么一个函数和一个普通的对象有什么差别呢？或者说函数之所以是函数，它的本质是什么呢？这里首先看一个 Python 内建函数 `callable`，看名字就能猜到这个函数能用来检测传进去的参数是不是可以调用的。下面这小段代码可以看到 `callable` 的用法

```python
    #!/usr/bin/env python3

    def hello():
        print('hello, world')

    foo = 3.1415926
    bar = 'linux'

    print('foo callable: ', callable(foo))
    print('bar callable: ', callable(bar))
    print('hello callable: ', callable(hello))
```

运行结果可以看出来，只有 `hello` 也就是这里唯一的函数是可以被调用的。

```
    foo callable:  False
    bar callable:  False
    hello callable:  True
```

这么看来只要我们搞一个 callable 的对象出来，就可以向函数那样调用喽？是的，只要我们给自己定义的类实现一个 `__call__` 方法，那么这个类实例化出来的对象就是 callable 的。例如这样

```python
    #!/usr/bin/env python3

    class SayHelloClass:
        def __call__(self):
            print('hello, world')

    say_hello_obj = SayHelloClass()
    print('say_hello_obj is callable: ', callable(say_hello_obj))
    say_hello_obj()
```

我们从执行结果可以看到，这个类实例化出来的对象在 `callable` 这里返回了 True，且被我们通过与调用普通函数相同的方式成功的调用了。所以我们可以说函数和普通的对象本质区别就在于是不是 callable 的，而是不是 callable 的则取决于类有没有实现一个 `__call__` 方法。

```
    say_hello_obj is callable:  True
    hello, world
```

# 0X03 可以被传递

既然函数都是对象了，那可以被当做参数或返回值传来传去也没什么特别的了。尤其是将函数作为参数传递，这在 Python 里有一个非常重要的概念是[装饰器](<https://www.liaoxuefeng.com/wiki/1016959663602400/1017451662295584>)，东西很多就不展开说了。

# 0X04 *args 和 **kwargs

`def foo(xxx, yyy, zzz, *args, **kwargs)` 这种函数定义方法不一定经常用，但是肯定见过不少了。就拿这个函数举例好了

```python
    #!/usr/bin/env python3

    def foo(xxx, yyy, zzz, *args, **kwargs):
        print(xxx, yyy, zzz)
        print(args)
        print(kwargs)

    print('------------------------------------')
    foo(1, 2, 3, 4, 5, 6)
    print('------------------------------------')
    foo(1, 2, 3, 4, 5, 6, 7, 8)
    print('------------------------------------')
    foo(1, 2, 3, 4, 5, 6, a=7, b=8, c=9)
    print('------------------------------------')
```

这里可以很容易看明白这两个参数的作用，但是有一个小问题你可能也许大概不知道，或者没尝试过

```python
    #!/usr/bin/env python3

    def foo(xxx, yyy, zzz, *args, aaa, bbb, ccc):
        print(xxx, yyy, zzz)
        print(args)
        print(aaa, bbb, ccc)


    foo(1, 2, 3, 4, 5, 6, aaa=7, bbb=8, ccc=9)
```

执行结果如下，这种把 `*args` 夹在中间的做法偶尔也会用得上。

```
    1 2 3
    (4, 5, 6)
    7 8 9
```

关于 `*args` 和 `**kwargs` 还有一个小知识点，多数人可能都知道。定义函数的时候 `args` 和 `kwargs` 两个名字只是大家习惯使用的，真正让语法生效的是前面的星号，原则上这两个单词随便用什么都可以，不过源于「代码是写给人看的」这个理念，还是建议任何时候都是用 `args` 和 `kwargs` 这两个普遍使用的单词拼写。

# 0X05 🪆套娃

套娃就很好理解了，也就是说可以在函数里定义函数，而且在函数里定义的函数还可以 return 到外面去。

```python
    #!/usr/bin/env python3

    def get_function(func_name):
        def say_hi():
            print('hi world')

        def say_hello():
            print('hello, world')

        def say_abaaba():
            print('a ba a ba a ba')

        if func_name == 'say_hi':
            return say_hi
        elif func_name == 'say_hello':
            return say_hello
        else:
            return say_abaaba

    func_1 = get_function('say_hi')
    func_1()

    func_2 = get_function('say_hello')
    func_2()

    func_3 = get_function('say_你好')
    func_3()
```

这种用法要注意每次调用 `get_function` 的时候都会定义 `say_hi/say_hello/sai_abaaba` 这三个函数，如果外层函数频繁被调用或者内部函数耗时耗资源比较多的话要慎用这种方式，尽可能考虑将子函数挪出去。或者可以这么说：除非你非常确信需要将其做成子函数并且理解其会怎样工作，否则就不要使用子函数。

# 0X06 λ lambda

说起 λ 这个符号，应该挺多人第一次见都是在 CS 里吧，当时我还以为这个字是入口的「入」，让我从那里进去呢😅

说正事，Python 中的 lambda 函数被称为「匿名函数」，当时刚接触的时候觉得这个名字取的真烂。后来才明白烂的不是名字，而是当时看到的那个示例

```python
    add = lambda a, b: a + b
    result = add(3, 5)
```

这个示例完了之后就告诉我说「匿名函数讲解完了」，我当时人都傻了，心里还在想这不是有名字吗？这名字不就是`add`吗？然后又回去翻看书上的例子，才觉得这个名字取的确实没问题。（当然这个例子是我自己写的，书里可不会有这么暴力的例子🐸）

```python
    #!/usr/bin/env python3

    data = {'shawn': 12, 'jack': 9, 'bluce': 22, 'robert': 19, 'frog': 999}

    result = sorted(data, key=lambda k: data.get(k))

    for k in result:
        print('{}: {}'.format(k, data.get(k)))
```

这个相对靠谱的例子中，就用 `lambda` 创建了个函数并且作为参数传了进去，然后实现了用字典中的值排序的功能。当然这种方法和用 `def` 定义一个标准函数再传进去没什么本质不同，但是优势就在于这种简单的函数定义方式可以直接写在参数里一行搞定。

```
    jack: 9
    shawn: 12
    robert: 19
    bluce: 22
    frog: 999
```

这里再次重申一遍「代码是写给人看的」，所以不要强行上 `lambda` ，如果函数逻辑稍微复杂点，甚至参数多一点都不建议使用 `lambda` ，毕竟 `lambda a, b, c, d, e: (func_1(a, b, c)[3].value + d).update(e)` 远远不如下面这个片段简单易懂

```python
    def foo(a, b, c, d, e):
        _, _, _, result = func_1(a, b, c)
        value = result.value +d
        return value.update(e)
```