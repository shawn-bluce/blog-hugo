---
title: "提升 git 新手效率的小技巧"
slug: "git-simple-tips"
date: "2018-08-22T14:26:00+0000"
lastmod: "2025-01-16T08:55:16+0000"
draft: false
tags:
  - "Git"
visibility: "public"
---
内容比较少，只是今晚翻看教程的时候发现的几个可以替换调我以前一些诡异操作的方法，将其整理贴出。

# 0X00 git blame

是谁在代码里下了毒？是谁用了一个超酷炫的方法解决了你解决不了的问题？当你想知道仓库中的某行代码是谁提交的，就可以使用这个方法。`git blame hello.py`可以看到hello.py这个文件所有行的提交人是谁，于何时提交的。而且这个命令最常用的是和`grep`合用，`git blame hello.py | grep prinft`(是谁写错了这个单词+_+)

```
    ^9544316 (Lars Schneider 2018-07-24 22:55:48 +0200   1) <p align="center">
    ^9544316 (Lars Schneider 2018-07-24 22:55:48 +0200   2)   <img src="https://s3.amazonaws.com/ohmyzsh/oh-my-zsh-logo.png" alt="Oh My Zsh">
    ^9544316 (Lars Schneider 2018-07-24 22:55:48 +0200   3) </p>
    ^9544316 (Lars Schneider 2018-07-24 22:55:48 +0200   4)
    ^9544316 (Lars Schneider 2018-07-24 22:55:48 +0200   5) Oh My Zsh is an open source, community-driven framework for managing your [zsh](https://www.zsh.org/) configuration.
    ^9544316 (Lars Schneider 2018-07-24 22:55:48 +0200   6)
    ^9544316 (Lars Schneider 2018-07-24 22:55:48 +0200   7) Sounds boring. Let's try again.
    ^9544316 (Lars Schneider 2018-07-24 22:55:48 +0200   8)
    ^9544316 (Lars Schneider 2018-07-24 22:55:48 +0200   9) _Oh My Zsh will not make you a 10x developer...but you might feel like one.__
    ^9544316 (Lars Schneider 2018-07-24 22:55:48 +0200  10)
    ^9544316 (Lars Schneider 2018-07-24 22:55:48 +0200  11) Once installed, your terminal shell will become the talk of the town _or your money back!_ With each keystroke in your command prompt, you'll take advantage of the hundreds of powerful plugins and beautiful themes. Strangers will come up to you in cafés and ask you, _"that is amazing! are you some sort of genius?"_
    ^9544316 (Lars Schneider 2018-07-24 22:55:48 +0200  12)
    ^9544316 (Lars Schneider 2018-07-24 22:55:48 +0200  13) Finally, you'll begin to get the sort of attention that you have always felt you deserved. ...or maybe you'll use the time that you're saving to start flossing more often. 😬
    ^9544316 (Lars Schneider 2018-07-24 22:55:48 +0200  14)
    ^9544316 (Lars Schneider 2018-07-24 22:55:48 +0200  15) To learn more, visit [ohmyz.sh](https://ohmyz.sh) and follow [@ohmyzsh](https://twitter.com/ohmyzsh) on Twitter.

```

> 上面的例子是`oh-my-zsh`中README的部分输出

# 0X01 git commit --amend

```sh
    vim xxx.py
    git add .
    git commit -m "naruto! sasuke!"
```

如果你不小心把刚刚的commit写错了，现在还来得及后悔。

如果你刚刚commit，还没有进行新的改动，那么可以使用`git commit --amend`来修改上一次的commit。输入命令回车之后会打开你的编辑器，最上面的就是本次提交的commit message，动手修改之后保存就可以了。如果commit之后改动了很多才想起来那也可以先`git add .`再`git stash`，将改动先临时存起来再执行`git commit --amend`，修改好了commit message之后再用`git stash pop`把刚刚的改动给pop出来。

> ps.在git里几乎一切都是来得及的，允许后悔药的存在。

# 0X02 git checkout -- filename

不管出于什么原因，大家都有可能需要删除掉对某一个文件的改动。如果是所有的改动都需要删除，那么可以简单的`git add .; git stash; git stash drop`丢弃这些改动。如果改动是多个文件，但是只有一个文件需要回退，那就可以使用`git checkout -- hello.py`来删除hello.py的改动。
