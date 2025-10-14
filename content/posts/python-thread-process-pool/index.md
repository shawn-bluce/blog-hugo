---
title: "Python中的线程、进程池"
slug: "python-thread-process-pool"
date: "2019-11-12T15:06:00+0000"
lastmod: "2025-01-17T01:49:01+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
# 0X00 线程池和进程池

多线程和多进程在平时编程中是挺常见的操作，不过控制进程和线程的数量是一件比较麻烦的事情。尤其是线程，之前在搜索到的关于线程池的内容多数都是“造轮子”，实际上Python已经给我们造好了这个轮子。文档在这里，甚至还是中文的<https://docs.python.org/zh-cn/3.7/library/concurrent.futures.html#module-concurrent.futures>

我这里简单的整理了一下，做个小样例展示出来方便查阅。这里就假装大家对Python有一定的了解，而且也对操作系统中的线程和进程有一些了解了。（哦对了，还需要了解一下GIL才行）

# 0X01 使用线程池

```python
    #!/usr/bin/env python

    import time
    from concurrent.futures import ThreadPoolExecutor


    # 定义一个用来测试的方法
    def test_func(num):
        time.sleep(1)
        res = num * num
        print(res)
        return res


    if __name__ == '__main__':
        # 构造一个可以容纳两个线程的线程池
        thread_pool = ThreadPoolExecutor(max_workers=2)

        with thread_pool as pool:
            for i in range(5):
                pool.submit(test_func, i)   # 将任务提交到线程池里
```

这段代码执行下来耗时`3s`大概，因为有两个线程在执行，所以第一次执行了两个任务，第二次两个，第三次一个。可以通过调整`range()`数量和`max_workers`来观察输出结果。

# 0X02 使用进程池

```python
    #!/usr/bin/env python

    import time
    from concurrent.futures import ProcessPoolExecutor


    def test_func(num):
        time.sleep(1)
        res = num * num
        print(res)
        return res


    if __name__ == '__main__':
        process_pool = ProcessPoolExecutor(max_workers=2)
        with process_pool as pool:
            for i in range(5):
                pool.submit(test_func, i)
```

可以看到跟上面线程池的方案比起来就只是把`ThreadPoolExecutor`换成了`ProcessPoolExecutor`而已。

# 0X03 通用的部分

其中`ProcessPoolExecutor`和`ThreadPoolExector`均接收参数`max_workers`，不过由于线程和进程的本质区别，所以还是要适当设置这两个值。默认情况下`max_workers`的值设置为自己的逻辑处理器个数，如果你的CPU是4核8线程的那就自动设置成8。在 Windows 上，max_workers 必须小于等于 61，否则将引发 ValueError。

`pool.submit()`中的参数是`submit(func, *args)`，所以把需要传递给`func`的参数逐个写在后面就好了。

`pool.submit()`后会返回一个`Future`对象，这个对象可以查看任务的执行情况：`cancel()`可以取消任务（如果任务还没开始的话）；`cancelled()`查看任务是否被取消了；`running()`任务是否在进行；`done()`任务是否执行完了。使用`result()`可以获得任务的结果，还未完成的任务会等待结果，可以使用`timeout`参数指定等待多少秒。

# 与Python无关的部分

  1. 使用多线程要注意不要开过多的线程，因为在线程中切换也需要资源，线程过多可能反而会影响效率；
  2. 进程不宜过多，防止系统负载过大；
  3. 使用多进程时要万分小心不要失控，因为进程数量一旦失控可能会导致系统宕机（相关内容可以搜索了解一下`fork炸弹`）。
