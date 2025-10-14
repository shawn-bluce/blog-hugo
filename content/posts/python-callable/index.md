---
title: "Python 内置函数：callable"
slug: "python-callable"
date: "2022-05-23T13:59:00+0000"
lastmod: "2025-01-16T09:53:17+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
# 0X00 换个方式定义函数

> 本篇内容不严格区分 function 与 method 🥹

我们都知道在 Python 中如何定义一个函数，只需要 `def foo(arg_1, arg_2, *args, **kwargs)`
就足够了。知道的稍微多一些呢可能知道「Python 中万物皆对象，所以函数的调用也只是调用了函数对象中的 `__call__` 方法」，所以我们可以尝试用这种方式调用一个函数

```python
    def say_hello():
        print('hello, world')

    say_hello.__call__()

    # output
    # hello, world
```

既然可以这样调用了，我们也就可以用类似的方法来定义一个假的 function，可以发现我们自定义了随便一个类，但是只要它实现了 `__call__` 方法就可以被当做函数一样调用

```python
    class Foo:
        def __call__(self):
            print('hello, world')

    foo = Foo()foo()

    # output
    # hello, world
```

# 0X01 callable

根据上面的方法可知我们可以用 `hasattr(obj, '__call__')` 来判断某个对象是不是函数，事实上我也确实在同事的代码里看到过这样用的。其实 Python 内置了一个名为 `callable` 的函数可以用，不过跟 `hasattr(obj, '__call__')` 并不完全一致

```python
    class Foo:
        pass


    class Bar:
        def __call__(self):
            pass

    func = lambda x : x


    print(hasattr(Foo, '__call__'), callable(Foo))
    # output: (False, True)

    print(hasattr(Foo(), '__call__'), callable(Foo()))
    # output: (False, False)

    print(hasattr(Bar, '__call__'), callable(Bar))
    # output: (True, True)

    print(hasattr(Bar(), '__call__'), callable(Bar()))
    # output: (True, True)

    print(hasattr(func, '__call__'), callable(func))
    # output: (True, True)
```

测试代码的第一行 `Foo` 类因为没有实现 `__call__` 方法所以 `hasattr` 返回的是 `False`，而它是一个类，调用它就会实例化一个对象出来，所以它是可调用的，所以 `callable` 就返回了 `True`；第二行 `Foo` 类也没有实现 `__call__` 方法所以 `hasattr` 返回的是 `False`，而且它又只是个普通对象，不是一个 `class` 所以导致 `callable` 也返回了 `False`；第三行因为 `Bar` 类实现了 `__call__` 方法所以 `hasattr` 和 `callable` 都返回了 `True`；第四行也同理；第五行本是一个函数，所以也都返回了 `True`。

需要注意的是官方文档提到「`callable` 返回了 `True` 的不一定真的能调用成功，但是返回 `False` 的一定不能成功」，比如你强行给某个类设置了一个 `__call__` 但是又不是函数，可能就会出现这样的问题。不过你非要这么写的话，小心被同事打死噢 🤔

```python
    class Foo:
        def __init__(self):
            self.__call__ = '???'

    foo = Foo()print(hasattr(foo, '__call__'))
    print(callable(foo))
    foo()

    # output
    # True
    # True
    # Traceback (most recent call last):
    #   File "hello.py", line 8, in <module>
    #     foo()
    # TypeError: 'str' object is not callable
```

> 相关的官方文档：<https://docs.python.org/3/library/functions.html#callable>
