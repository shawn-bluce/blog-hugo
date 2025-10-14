---
title: "Python 中的 pyc 文件"
slug: "python-pyc-simple"
date: "2017-12-23T15:44:00+0000"
lastmod: "2025-01-17T01:46:16+0000"
draft: false
tags:
  - "Python"
visibility: "public"
---
我们在编写Python程序的时候会发现在我们的目录中可能会出现与源代码同名的pyc文件生成，比如有一个源码文件是`hello.py`那么可能会生成一个`hello.pyc`文件出来．这个pyc文件是Python的字节码文件，就类似于Java中的`hello.class`一样．

Python虽然是解释性语言，但还是可以有一个编译的过程，只不过是编译成字节码文件罢了．如果我们的源码带有包含关系，比如源码`a.py`里面`import`了源码文件`b.py`，那么在执行`python a.py`的时候就会将`b.py`编译成`b.pyc`．

下次再运行`python a.py`的时候解释器会检查`b.py`与`b.pyc`的修改时间，如果一致就代表源码没有修改过，那么就可以直接调用`b.pyc`来更快的执行程序，如果时间不同那就证明`b.py`被修改过，则会重新编译`b.py`．

其中`pyc`文件的主要作用是用来加快程序执行速度的，虽然编译出来的`pyc`是二进制文件，不能看到内部的内容，但是还是不要把这种方式作为保护源码的方法．因为这种`pyc`文件是可以轻易被反编译的，有很多开源库可以轻松的反编译`pyc`文件，甚至都有web程序来反编译`pyc`文件，比如这个[tool.lu](<https://tool.lu/pyc/>)就可以通过上传文件`pyc`文件的方式反编译．
