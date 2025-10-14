---
title: "Python 上下文管理器"
slug: "python-context-manager"
date: "2020-07-18T05:27:00+0000"
lastmod: "2025-01-16T09:57:01+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
# 0X00 使用 with as 语法

我们写程序经常会操作文件，我们都知道写文件要 `open/write/close` ，尤其是 `close` ，没有的话文件就会出问题（有些内容在缓存里，没写入磁盘）。不过我们现在写文件应该没什么人这样写了，都是用`with open('filename', 'w') as f`的方式来操作文件了。如果说这样做的好处，那多数人都会说“不用手动关闭文件了”，错肯定没错的。

![](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/20200718142750.png)

上下比起来，上面的方式不仅多了一点点代码，而且随着中间逻辑代码变多，很可能会导致最后忘记 `close`，从而引发 bug。

# 0X01 这就是上下文管理器

上面 `with xxx as xxx` 的调用方式就是在调用上下文管理器。简单来说上下文管理器就是：在执行你编写的代码（with xxx as xxx后面那坨）之前，**操作一波** ；再在你编写的代码执行完后，**操作一波** 。我们简单理解一下就是在 `with open('file_name', 'w') as f` 内层缩进的代码执行完成后自动帮你执行了`f.close()`（当然没这么简单，有兴趣可以去看一下 open 的源码，但是大体逻辑是这样的）。

# 0X02 自己实现一个

说了半天，咱们自己来实现一个上下文管理器好了。自己实现一个上下文管理器跟实现一个其他东西不太一样，不用继承任何东西，就像实现一个迭代器一样，只需要满足自己的协议（也就是上下文管理器协议）就可以。而且好在这个协议极其简单，实现一个类只需要满足满足两个方法就可以：`__enter__` 和 `__exit__`。我们先来写个 demo 试试看。

```python
    #!/usr/bin/env python3

    class CustomProtocolConnection:
        def __init__(self, host, port):
            print('假装开始连接, ', host, port)

        def __enter__(self):
            print('进入了 __enter__')
            return self

        def __exit__(self, exception_type, exception_value, exception_traceback):   # 这个定义是固定的，必然接收三个参数
            print('进入了 __exit__')
            self.close()    # 没有定义，只是断开连接
            return True

        def push_data(self, data):
            print('假装在推数据：', data)

        def pull_data(self):
            print('假装收到了数据：', 'hello,world')


    if __name__ == '__main__':
        with CustomProtocolConnection('127.0.0.1', '2333') as conn:
            conn.push_data('hello, world')
            conn.pull_data()
```

然后我们来看一下执行结果

![](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/20200718142514.png)

首先进行了常规的实例化；实例化之后执行了`__enter__`；然后执行我们自己编写的代码块；退出代码块之后执行了`__exit__`。我们可以看到上面定义`__exit__`的时候带了三个参数，看名字也看出来了，第一个参数是异常类型、第二个是异常的值（也就是错误消息）、第三个是异常的错误栈。如果我们在 `with xxx as xxx` 下面的代码块中出现异常了，那么异常会被捕获并传递到`__exit__`这里来，你可以根据情况来处理。这里不再展示具体如何处理异常了，大家可以手动触发异常然后在`__exit__`里打上断点来调试一下，很容易就能知道这里是怎么用的了。

实现一个上下文管理器不一定非要是类，一个函数照样可以（废话，`with open() as f` 不就是一个函数嘛）。我们可以来编写这样一个函数

```python
    #!/usr/bin/env python3

    import contextlib


    class CustomProtocolConnection:     # 一个本来就有的类
        def __init__(self, host, port):
            print('假装开始连接, ', host, port)

        def push_data(self, data):
            print('假装在推数据：', data)

        def pull_data(self):
            print('假装收到了数据：', 'hello, world')



    @contextlib.contextmanager      # 用 contextmanager 装饰器使这个方法成为上下文管理器
    def connect_2_server(host, port):
        conn = CustomProtocolConnection(host, port)
        print('进入了 __enter__')  # 并不是真正的 __enter__ 方法，但是有同样的效果
        yield conn
        print('进入了 __exit__')   # 同理，并不是真正的 __exit__ 方法，但是又相同的效果


    with connect_2_server('127.0.0.1', '2333') as conn:
        conn.push_data('hello, world')
        conn.pull_data()
```

一个带有 `yield` 的函数是一个生成器，不过因为 `contentlib.contentmanager` 装饰器的作用，使它现在是一个上下文管理器了。以 `yield` 为界限，`yield` 之前的内容可以理解成是 `__enter__` 方法，后面的可以理解成是 `__exit__` 方法，由 `yield` 返回的那个值就是我们 `with connect_2_server as conn` 的 conn 了。我们来看一下这坨代码的运行结果，可以看到跟上面的效果是一样的。这两种方式其实是适用于不同的场景的，第一种直接编写 class 的方式适用于从零开始实现一个东西，这种就可以直接将其定义为上下文管理器，用起来比较方便；第二种通过装饰器实现一个额外的 function 的方式适用于在现有的代码块基础上实现上下文管理器，可以做到不侵入现有代码还实现所需功能的需求。

![](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/20200719105451.png)

# 0X03 什么时候用

现在搞明白上下文管理器是什么了，也知道自己怎么才能实现一个上下文管理器了，那么什么时候才会用到这种东西呢？其实我们的操作很多时候都是可以用到的，只不过这个东西从来都不是必须的，很多情况下大家都绕开或者用了更麻烦一点的方式实现了。

首先，就像上面的自定义协议的通信一样，先要连接最后断开的情况就可以使用这种方式；比如要发起一大波 HTTP 请求，但是需要登录，就可以用这种方式实现自动登录和自动注销；再比如做数据统计或者导出，开始前要将数据整理一波，统计导出结束后再将结果邮件发送到指定邮箱，这种也是可以的。
