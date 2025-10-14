---
title: "一些由 int 方法引出的小知识点"
slug: "python-int"
date: "2022-05-22T13:58:00+0000"
lastmod: "2025-01-16T09:54:34+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
# 0X00 可以被强制转换的自定义类

但凡写过 Python
的人应该都用过`int()`这个函数了，而且也都知道这个是将其他类型转换成`int`类型的内置方法，稍微用的多一点的还会知道这个方法如果传入不能被强制转换的数据时会抛出`TypeError`的异常。那你知道如何让自己定义的类可以被强制转换吗？

```python
    #!/usr/bin/env python3


    class A:
        def __int__(self):
            return 233


    a = A()
    print(int(a))

    # output
    # 233
```

而且按照官方文档来说的话，如果你的`class`定义了`__int__()`方法，则`int(your_obj)`则会返回`__int__()`的值，如果定义了`__index__()`则会返回`__index__()`，如果定义了`__trunc__()`也会返回`__trunc__()`。当然是有优先级的，优先级`int

> index > trunc`，可以使用如下代码分别注释这些方法测试一下

```python
    #!/usr/bin/env python3


    class A:
        def __int__(self):
            return 1

        def __index__(self):
            return 2

        def __trunc__(self):
            return 3


    a = A()
    print(int(a))
```

# 0X01 int 的第二个参数

那你知道它其实还能接收第二个参数吗？其实 `int()`
方法可以接受第二个参数的，也就是用于进制转换的参数。换言说就是可以用内置的`int()`方法将其他进制的字符串数据转换成10进制

```python
    #!/usr/bin/env python3


    print(int('12345'))  # 将字符串格式10进制数字转为整型
    print(int('12345', base=10))  # base 默认就是10
    print(int('12345', base=8))   # 8进制的12345转成10进制
    print(int('FFFFF', base=16))  # 16进制的转成10进制
    print(int('0XDEADBEEF', 16))  # 当然可以带对应的0X前缀
    print(int('011111', 2))       # 将2进制的数字转成10进制

    # output
    # 12345
    # 12345
    # 5349
    # 1048575
    # 3735928559
    # 31
```

# 0X02 hex、 bin 等其他转换方法

上面提到了进制转换，这里也就顺便说一下这两个方法好了。其中`hex`可以将整型数字转成0x开头的16进制字符串，`bin`可以将整形数字转成0b开头的2进制字符串

```python
    #!/usr/bin/env python3


    print(hex(12345))
    print(hex(3735928559))
    print(bin(12345))
    print(bin(23333))
```

还有几个之前从来不知道，这次写博客才在官方文档看到的用法，不仅可以控制大小写还能控制是否展示0X这种标记

```python
    In [1]: '%#x' % 255, '%x' % 255, '%X' % 255
    Out[1]: ('0xff', 'ff', 'FF')

    In [2]: format(255, '#x'), format(255, 'x'), format(255, 'X')
    Out[2]: ('0xff', 'ff', 'FF')

    In [3]: f'{255:#x}', f'{255:x}', f'{255:X}'
    Out[3]: ('0xff', 'ff', 'FF')

    In [4]: format(14, '#b'), format(14, 'b')
    Out[4]: ('0b1110', '1110')

    In [5]: f'{14:#b}', f'{14:b}'
    Out[5]: ('0b1110', '1110')

    In [6]: '%#o' % 10, '%o' % 10
    Out[6]: ('0o12', '12')

    In [7]: format(10, '#o'), format(10, 'o')
    Out[7]: ('0o12', '12')

    In [8]: f'{10:#o}', f'{10:o}'
    Out[8]: ('0o12', '12')
```

> 官方文档参考：

<https://docs.python.org/zh-cn/3/library/functions.html#int>

> <https://docs.python.org/zh-cn/3/library/functions.html#hex>

> <https://docs.python.org/zh-cn/3/library/functions.html#bin>

> <https://docs.python.org/zh-cn/3/library/functions.html#oct>
