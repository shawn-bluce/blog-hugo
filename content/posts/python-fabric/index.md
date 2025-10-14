---
title: "Python 自动化运维与远程部署：fabric"
slug: "python-fabric"
date: "2017-12-10T13:27:00+0000"
lastmod: "2025-01-16T10:04:51+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
# 0X00 安装fabric

使用pip可以轻松地安装fabric

```sh
    pip install fabric
```

# 0X01 初次调用

在当前目录下创建一个名为`fabfile.py`的文件，填写文件内容如下：

```python
    # coding=utf-8
    import fabric


    def test():
        print 'hello,world'
```

然后在当前目录下执行命令`fab test`就可以看到一条`hello,world`输出了。

# 0X02 浅显的道理

根据上面简单的例子可以看出来fab命令执行的时候会默认找到当前目录下的`fabfile.py`文件，找到后会用fab命令的参数去匹配`fabfile.py`中的函数名，执行相应的功能。

> 实际上当前目录可以没有`fabfile.py`，如果当前目录的上级目录中有`fabfile.py`是会采用上级目录中的`fabfile.py`的。而且文件名也不一定用`fabfile.py`，假设取了一个名为`asdf.py`的文件，那么只需要执行`fab -f asdf.py`就可以采用这个`fabfile`了。

# 0X03 执行本地命令

作为一个可以用来自动化运维和远程部署的库，运行本地命令是一个必不可少的功能。

```python
    # coding=utf-8

    from fabric.api import local


    def list_home_dir():
        local('ls ~/.')
```

这里的`local`方法就是执行本地命令，并将输出打印出来。然后运行`fab list_home_dir`就可以看到自己`~`目录下的文件们了。当然，也可以带参数的。

```python
    def cp_file(file_a, file_b):
        local('cp %s %s' % (file_a, file_b))
```

调用这个方法的时候可以通过`fab cp_file:"file_a=~/hello.py,file_b=~/hello_b.py"`这个命令将`~/hello.py复制到hello_b.py`。

# 0X04 执行远程命令

fabric用处最大的一点就是远程执行命令了。使用下面这段代码，运行`fab list_home`后`fabric`会使用你提供的登录信息通过`SSH`登录到远程机器上执行`ls ~/.`的命令。其中`run()`就是在远程机器上执行命令。

```python
    # coding=utf-8

    from fabric.api import run, env

    env.hosts = ['qcloud.just666.cn', ]
    env.user = 'root'
    env.password = '5L2g5b2T5oiR5YK75ZWK'

    def list_home():
        run('ls ~/.')
```

# 0X05 多台机器执行相同的命令

有的时候我们有多台机器，需要执行相同的命令，这种时候就可以用`env.passwords`来解决问题。通过`hosts`指定用户名和主机，用`passwords`指定密码，就可以同时登录到多台机器上了。

```python
    #!/usr/bin/env python
    # -*- coding: utf-8 -*-

    from fabric.api import env, run

    # 指定主机
    env.hosts = [
        'root@qcloud.just666.cn',
        'root@aliyun.just666.cn',
        'root@aws.just666.cn',
    ]

    # 指定密码
    env.passwords = {
        'root@qcloud.just666.cn': '5L2g5b2T5oiR5YK75ZWK',
        'root@aliyun.just666.cn': '5L2g5b2T5oiR5YK75ZWK',
        'root@aws.just666.cn': '5L2g5b2T5oiR5YK75ZWK',
    }

    def hello():
        run('ls ~/.')
```

其实我也不是很懂，既然`env.passwords`都已经制定了用户名，主机和密码为什么还要用hosts再指定一次呢？不是很懂，注释过`hosts`，就不能用了。

# 0X06 多台机器执行不同命令

不过通常情况下我们是有不止一台远程机器的，要不然也不会需要什么自动化了。那么假设我们有三台机器，分别是`mysql/apache/nginx`这三个服务，那么我们可以这么写脚本。这样我们可以通过`fab start_firewalld`启动三台机器的防火墙，使用`fab start_mysql/start_httpd/start_nginx`分别启动在这三台机器上的三个服务。

```python
    #!/usr/bin/env python
    # -*- coding: utf-8 -*-

    from fabric.api import env, run, roles

    env.hosts = [
        'root@mysql.just666.cn',
        'root@apache.just666.cn',
        'root@nginx.just666.cn',
    ]

    env.passwords = {
        'root@mysql.just666.cn': 'zhangHAO8',
        'root@apache.just666.cn': '5L2g5b2T5oiR5YK75ZWK',
        'root@nginx.just666.cn': '5L2g5b2T5oiR5YK75ZWK',
    }

    env.roledefs = {
        'all': [
            'root@mysql.just666.cn',
            'root@apache.just666.cn',
            'root@nginx.just666.cn',
        ],
        'mysql': ['root@mysql.just666.cn', ],
        'apache': ['root@apache.just666.cn', ],
        'nginx': ['root@nginx.just666.cn', ],
    }

    @roles('all')
    def start_firewalld():
        run('systemctl start firewalld')

    @roles('mysql')
    def start_mysql():
        run('systemctl start mysql')

    @roles('apache')
    def start_httpd():
        run('systemctl start httpd')

    @roles('nginx')
    def start_nginx():
        run('systemctl start nginx')
```

# 0X07 并发任务

通常情况下`fabric`是穿行执行任务的，假设有100台机器要执行相同的命令，虽然我们批量化了，但是他们依旧是串行的，导致效率比较低。这种时候我们可以采用`@parallel`装饰器来使方法变成并行的。比如有一个方法`def test_speed`用来测试机器的网速，需要持续一分钟才行，并且有很多台机器，那么这个`@parallel`就可以发挥作用了。

```python
    from fabric.api import parallel

    @parallel
    def test_speed():
        run('test_speed')
```


此时这些任务就是并行的了，会快很多。不过如果真的有非常多的机器要并行，那么fabric可能扛不住，可以给并行数量设置一个上限`@parallel(pool_size=10)`，这样就是最高10个并行任务了。

# 0X08 传文件

文件传输用的是`scp`的原理，分成两个方法，分别是`get/put`，用法也非常简单就像普通的`cp`一样。`get('remote_file', 'local_file')`和`put('local_file', 'remote_file')`

```python
    from fabric.api import env, get, put

    env.hosts = ['qcloud.just666.cn', ]
    env.user = 'root'
    env.password = '5L2g5b2T5oiR5YK75ZWK'

    def get_vimrc():
        get('~/.vimrc', '~/.vimrc')

    def put_vimrc():
        put('~/.vimrc', '~/.vimrc')
```

# 0X09 切目录

在`fabric`中切目录要配合Python的`with`语法使用。共有`cd/lcd`这两种切目录方法，对应的是远程目录和本地目录。

```python
    from fabric.api import env, cd, lcd

    env.hosts = ['qcloud.just666.cn', ]
    env.user = 'root'
    env.password = '5L2g5b2T5oiR5YK75ZWK'

    def list_local_dir():
        with lcd('/'):  # 切到本地根目录
          local('ls') # 在目录下执行命令
        with lcd('~'):  # 切到本地主目录
          local('ls') # 在目录下执行命令

    def list_remote_dir():
        with cd('/'):   # 切到远程根目录
          run('ls') # 在目录下执行命令
        with cd('~'):   # 切到远程主目录
          run('ls') # 在目录下执行命令
```

# 0X0A PATH

有的时候我们需要临时添加PATH，可以通过这种方法。

```python
    from fabric.api import env, run, path

    env.hosts = ['qcloud.just666.cn', ]
    env.user = 'root'
    env.password = '5L2g5b2T5oiR5YK75ZWK'

    def hello():
        with path('/home/shawn/.envs/study_django/bin/'): # 添加目录到PATH中
            run('echo $PATH') # 查看新的PATH
        run('echo $PATH') # 退出去后PATH也被删除了
```

# 0X0B 环境变量

如果要临时修改环境变量的话可以这样子：

```python
    from fabric.api import env, run, local, shell_env

    env.hosts = ['qcloud.just666.cn', ]
    env.user = 'root'
    env.password = '5L2g5b2T5oiR5YK75ZWK'

    def hello():
        with shell_env(JAVA_HOME='/opt/oracle/java'):
            run('echo $JAVA_HOME')  # 远程机器的环境变量被修改了
            local('echo $JAVA_HOME')  # 本地的环境变量也被临时修改了。
```

> 修改只是暂时的，退出`with`之后就会恢复原状的。

# 0X0C 带颜色输出

有的时候为了便与查看输出结果，我们可能会需要用不同颜色的字标示不同的内容。比如用红色标示失败，黄色标示警告，绿色标示成功等。`fabric`也可以轻松实现这个功能的。

```python
    from fabric.colors import *

    def test_color():
        print green("OK.")
        print yellow("Warning")
        print red("Error")
```