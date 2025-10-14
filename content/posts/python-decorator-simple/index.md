---
title: "Python 装饰器"
slug: "python-decorator-simple"
date: "2019-08-22T13:18:00+0000"
lastmod: "2025-01-16T10:00:26+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
# 0X00 给一个方法计时

现在我们有一个需求，需要给程序中的一部分方法计时，以监控他们执行完具体用了多久。那么在没有装饰器的情况下我们会写出类似这样的代码：

```python
    import datetime

    def foo():
        pass    # some code


    start = datetime.datetime.now()
    foo()
    end = datetime.datetime.now()
    print((end - start).seconds)
```

# 0X01 引入装饰器

如果只是临时给一个方法使用也不是不行，但是如果我们需要监控大量的方法呢？众所周知Python中`function`也是可以作为参数传递的，那来看一下下面这种写法呢

```python
    import datetime


    def timer(func):
        def run_func(): # 方法内部将之前计时的功能封装起来
            start = datetime.datetime.now()
            result = func() # 得到参数方法的返回值
            end = datetime.datetime.now()
            print((end - start).seconds)
            return result   # 将真正的结果返回给调用者

        return run_func   # 调用内部方法，进行计时


    def foo():
        pass    # some code


    def bar():
        pass    # some code


    # 将方法作为参数发送给 timer()
    timer(foo)()
    timer(bar)()
```


**因为上面timer(foo)返回的结果是一个`func`，所以后面需要再加一对括号来调用这个方法**

这种方法只写了一次计时的逻辑，但是可以给任意一个方法使用，其实这时候`def timer(func)`已经是装饰器了，下面调用的方法`timer(foo)/timer(bar)`也是正确的装饰器使用方法。那是不是觉得和常见的装饰器使用不太一样呢？其实常见的`@timer`用法是Python中提供的一种语法糖。

# 0X02 @语法糖

其实上面的代码就可以直接使用@语法糖了，具体用法是这样：

```python
    @timer
    def foo():
        pass    # some code


    foo()
```

语法糖实际只是便于我们编码的一种设计，按理说一切被称为语法糖的东西都不是必须的。比如说这里的语法糖完全可以用上面的方法来应用装饰器，但是为什么还是设计了这个语法糖呢？我们对比下面两种方法的调用，我们假设`get_data_from_server/db`是一个耗时较久的方法：

```python
    def get_data_from_server():
        pass


    @timer
    def get_data_from_db():
        pass


    timer(get_data_from_server))()

    get_data_from_db()
```

这两种方法看起来明显是后面`get_data_from_db`的用法更易读。所以这处语法糖的出现能大幅度的提升代码的可读性，更能提升维护性质：当代码中调用了1W次非语法糖形式的装饰器计时时，取消计时就要修改1W行代码；如果用了@语法糖，那就只需要将方法定义处的`@timer`注释掉就可以了。

# 0X03 传参

“那要传参咋搞哇？” “这么搞”

```python
    #!/usr/bin/env python
    # coding=utf-8

    import time
    import datetime


    def timer(func):
        def run_func(secs): # 这儿接受参数
            start = datetime.datetime.now()
            result = func(secs) # 这儿把参数搞进去
            end = datetime.datetime.now()
            print((end - start).microseconds)
            return result

        return run_func


    @timer
    def foo(secs):
        time.sleep(secs)


    @timer
    def bar(secs):
        time.sleep(secs)


    if __name__ == '__main__':
        foo(3)  # 和不使用@timer时候的timer(foo)(3)是一样的
        bar(5)  # 和不使用@timer时候的timer(bar)(5)是一样的
```

“那我要是有好几个参数呢，怎么搞？” “就还是这么搞啊”

```python
    def timer(func):
        def run_func(*args): # 这儿接受参数，几个都行
            start = datetime.datetime.now()
            result = func(*args) # 这儿把参数搞进去，几个都行
            end = datetime.datetime.now()
            print((end - start).microseconds)
            return result

        return run_func


    @timer
    def foo(start_secs, end_secs):
        time.sleep(random.randint(start_secs, end_secs))


    foo(3, 5)
```