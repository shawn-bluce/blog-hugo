---
title: "属于 Python 程序员的小技巧"
slug: "python-developer-cool"
date: "2022-11-15T14:23:00+0000"
lastmod: "2025-01-16T09:57:48+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
# 0X00 这是一个描述

下面介绍几个我自己常用的小技巧，均可以在日常工作中给自己带来一些小小的便利🤪

# 0X01 临时 web 服务器

如果你电脑上装了 Python3 则可以使用 `python3 -m http.server --bind 0.0.0.0 2333` 这个命令在当前目录启动一个简单的 web 服务，监听在 0.0.0.0:2333 上。这样一来别人就可以访问你的 ip 来下载当前目录下的文件了。不过使用这个方法的时候要注意自己当前的工作目录哈，不要傻乎乎的在自己的 `$HOME` 下面用这个命令，小心别人下载你的隐私数据喔。

不过如果你搞不懂什么是「监听、0.0.0.0、端口」的话，还是先去搞一下计算机网络吧。

![python_http](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/11/15/pythonhttp.png)

# 0X02 使用 pdb/ipdb 调试脚本

我们都知道用 `pdb` 模块可以逐行调试 Python 脚本，只需要在脚本里 `import pdb` 然后在需要打断点的地方加上 `pdb.set_trace()` 就可以了。但其实 Python 还有一个三方库叫做 `ipdb` 是 `pdb` 的升级版，应该各位也都知道吧，通过简单的 `pip install` 就可以装好。不论是 `pdb` 还是 `ipdb` 其实都有这个用法：不修改脚本内容，仅通过 `python3 -m ipdb script_filename.py` 来从头调试脚本。并且会在每次脚本结束之后从头开始，直到手动退出为止。

tips: pdb/ipdb 运行时可以使用 `b line_number` 来在指定行打上断点，然后再 `c` 跳到下一个断点处从而方便调试。

# 0X03 指定版本

我们都知道可以用 virtualenv 之类的东西处理 Python 的虚拟环境，但是还有这么一种简单粗犷的方式也可以凑合一用（尤其是同时存在 Python 2.7/3.7/3.8/3.9... 的这种环境）。

使用 `python2.7 -m pip install requests` 或者 `python3.9 -m pip install xxx` 可以指定具体的 Python 版本，如果你有多个虚拟环境也可以直接指定到 Python 的二进制文件，再通过 `-m pip` 的方式将所需的库安装到正确的位置~

# 0X04 一行流

这个「一行流」比较骚气，相当于把 Python 当成普通命令来用，简单的说就是 `python -c "xxxxx"` 可以在一行命令里执行一个 Python 脚本。例如有下面这些用法（其实就是把脚本写成一行）

比如说使用 `python3 -c "import json; print(json.dumps(json.load(open('data.json')), indent=4))"` 这样的方法可以直接将 json 文件格式化输出。（当然了这个需求其实有 `jq` 命令可以做的更好，只是这里举个例子而已）

![one_liner](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/11/15/oneliner.png)

具体这个一行流的用法有多强，那就要看各位的脑洞了，唯一一个需要提示的就是 Python 其实是允许使用分号来终止一个语句的，类似 C 那样，所以才使得一行流可以实现~
