---
title: "git 初步使用经验"
slug: "git-simple"
date: "2017-11-12T07:24:00+0000"
lastmod: "2025-01-16T08:53:55+0000"
draft: false
tags:
  - "Git"
visibility: "public"
---
# 0X00 怎样正确使用分支

通常情况下一个git仓库要保持三个及以上的分支，基本的分支明明如下：

name | function
---|---
master | 正常运行的稳定版本
develop | 正常运行的开发版
feature | 添加新功能的分支
hotfix | 紧急修复bug的分支

如果你已经fork了一份代码到自己本地，当你想添加一个新功能比如「用户管理」的时候，就应该先换到`develop`分支，然后由这个分支创建一个新的名为`feture_add_usermanager`的分支。在新分支里编写代码后将代码提交一个PullRequest到自己的`develop`分支，合并起来后再提交一个PullRequest到团队的仓库中，等待团队其他成员review后就可以正式将代码合并到团队的`develop`中了。等下一次发布新版本的时候就可以将团队的`develop`分支合并到团队的`master`分支中了了。

如果中途项目出现了严重bug(不能登录)需要即使修改上线，那就从自己的`master`分支上新建一个名为`hotfix_cantlogin`的分支，修改完后直接提交PullRequest到团队的`master`分支，review且合并后就可以将该分支删除了。

# 0X01 合并多次commit

有的时候会出现这么一种情况：连续的n次commit解决的都是同一个问题，为了让最终的提交记录清晰明了，可以将这几次的commit合并为一次。看下面的实例：

```
    commit bc83e15e3245ad4064fdd9fe2bf105252d0c51fc (HEAD -> develop)
    Author: shawn <shawnbluce@gmail.com>
    Date:   Mon Nov 6 00:52:43 2017 +0800

        fix usermanager

    commit affc0acb9a3de30288f144ae2d0b28a9fd60af4b
    Author: shawn <shawnbluce@gmail.com>
    Date:   Mon Nov 6 00:52:33 2017 +0800

        fix usermanager

    commit fcf13d712bd3dbd483442ffd316741e04acaaa3e
    Author: shawn <shawnbluce@gmail.com>
    Date:   Mon Nov 6 00:52:23 2017 +0800

        fix usermanager

    commit 2812e2928913e9b3a7650e389602b8bb10ca388b
    Author: shawn <shawnbluce@gmail.com>
    Date:   Mon Nov 6 00:52:08 2017 +0800

        fix usermanager

    commit f7f273d9c36d748183ac3e6a90f06ff14ed42f95
    Author: shawn <shawnbluce@gmail.com>
    Date:   Mon Nov 6 00:51:52 2017 +0800

        add user_manager
```

最近的四次提交内容都是一样的，修改了四次bug最终才解决了问题，因为最近四次的commit是相同含义的修改，所以最好合并在一起，可以使用这个方法

```
    # 最后接的id是你合并的这些commit之前的一个
    git rebase -i f7f273d9c36d748183ac3e6a90f06ff14ed42f95

    ＃ 输入命令之后到vi环境的界面
    pick 2812e29 fix usermanager
    pick fcf13d7 fix usermanager
    pick affc0ac fix usermanager
    pick bc83e15 fix usermanager

    # 我们将其修改为
    pick 2812e29 fix usermanager
    squash fcf13d7 fix usermanager
    squash affc0ac fix usermanager
    squash bc83e15 fix usermanager

    # 保存退出，进入commit message修改的界面
    这里会展示出合并的这几次commit的message，我们选择性的修改一下将这几次的commit message整合一下，继续保存。这次保存之后就相当把这几次的commit合并成一次了。
```

> 注意，如果你是从仓库中clone下来的项目，那么有可能在你合并完几个commit后再`git push`会提示错误。假设以前是有100次commit的，现在你合并了最后的三次，再push，就会和远端的仓库出现分歧。一般情况下都是在确认自己的代码没问题之后，使用`git push -f`的方式强行把代码推上去，这样就会覆盖掉最后的几次commit，以你新push的代码为基准了。

# 0X02 迁移一次commit

有的时候我们需要将某个分支中的某次提交复制到另一个分支，具体使用情景有很多。这里展示一个使用样例：


    git log 查看master分支的commit，只有这一次
```
    commit 2959f92753d84bff5b125b15cd9497c4d8dff637 (HEAD -> master)
    Author: shawn <shawnbluce@gmail.com>
    Date:   Sun Nov 12 16:09:07 2017 +0800

        完成XXX功能

    切换到dev分支查看commit，有四次
    commit bba77c7faf7821802b108688a17d8e6655975b7e (HEAD -> dev)
    Author: shawn <shawnbluce@gmail.com>
    Date:   Sun Nov 12 16:11:47 2017 +0800

        整理XXX

    commit 9001f8250ce47a067c9af50bf62af025b872c3bc
    Author: shawn <shawnbluce@gmail.com>
    Date:   Sun Nov 12 16:11:37 2017 +0800

        添加XXX

    commit 6e4171ad632a7627be08ec7c9afaf64f55ccc98d
    Author: shawn <shawnbluce@gmail.com>
    Date:   Sun Nov 12 16:11:21 2017 +0800

        修复XXX

    commit 2959f92753d84bff5b125b15cd9497c4d8dff637 (master)
    Author: shawn <shawnbluce@gmail.com>
    Date:   Sun Nov 12 16:09:07 2017 +0800

        完成XXX功能

    如果此时我想将那个“添加XXX”的commit移到master分支，可以使用`git cherry-pick`
    可以看到“添加XXX”的那次commit的ID为`6e4171ad632a7627be08ec7c9afaf64f55ccc98d`
    我们复制这个ID，切到master分支，使用`git cherry-pick commit_id`
    shawn in ~/test on master λ git cherry-pick 6e4171ad632a7627be08ec7c9afaf64f55ccc98d
    [master f77b496] 修复XXX
     Date: Sun Nov 12 16:11:21 2017 +0800
     1 file changed, 0 insertions(+), 0 deletions(-)
     create mode 100644 b
    shawn in ~/test on master λ git log
    commit f77b49677efb1ba967dbc323332e2f8495fdbca1 (HEAD -> master)
    Author: shawn <shawnbluce@gmail.com>
    Date:   Sun Nov 12 16:11:21 2017 +0800

        修复XXX

    commit 2959f92753d84bff5b125b15cd9497c4d8dff637
    Author: shawn <shawnbluce@gmail.com>
    Date:   Sun Nov 12 16:09:07 2017 +0800

        完成XXX功能
```
    这样一个commit就复制过来了。如果有冲突的话需要将冲突解决完才能合并。