---
title: "使用 git stash save 将暂存区命名"
slug: "git-stash"
date: "2018-03-04T16:23:00+0000"
lastmod: "2025-01-16T08:57:25+0000"
draft: false
tags:
  - "Git"
visibility: "public"
---
# git stash save/apply/pop

在用git的时候经常会有需要临时切分支等操作，但是如果当前工作区进行了修改就不能直接切分支。这时候呢就得把当前的代码暂存起来，可以这么操作：

```sh
    git add .
    git stash
```

这样就吧上次commit到现在的修改都暂存起来了，可以使用`git stash show`来查看暂存区。我以前就是这样的，每次由两个或是两个以上的stash之后就蒙圈了，不知道那个stash做了哪些改变。虽然`git stash show`可以看到每个stash修改了哪些文件，但是还是不能准确的定位到自己需要的stash。

后来发现`git stash`后面还能继续接参数，这里得感谢`git plugin for oh-my-zsh`。当临时保存一些修改的时候可以这样：`git stash save "fix:xxxxx"`，有多个stash的时候也可以用`git stash show`来看到每个stash的备注，就方便多了。

```sh
    git on  new_branch [$?] via simple took 2s
    ➜ git stash show stash@\{0\}
    stash@{0}: On new_branch: feature:xxxxx (38 seconds ago)               stash@{3}: On new_branch: create new file named hello.py (4 days ago)
    stash@{1}: On new_branch: fix:xxxxx (53 seconds ago)                   stash@{4}: WIP on master: 75e1918 add a (7 weeks ago)
    stash@{2}: On new_branch: create file zzz (4 days ago)
```

其中`git stash pop`是应用一个stash，并删除这个stash。`git stash apply`是只应用不删除。

```sh
    git on  new_branch [$] via simple
    ➜ git stash pop stash@\{0\}
    On branch new_branch
    Changes to be committed:
      (use "git reset HEAD <file>..." to unstage)

    	new file:   a

    Dropped stash@{0} (163b1e2391b4c8bd792701ac4318d928e0e12556)

    git on  new_branch [$] via simple
    ➜ git stash apply stash@\{1\}
    On branch new_branch
    Changes to be committed:
      (use "git reset HEAD <file>..." to unstage)

    	new file:   a
```


善用git可以大幅提升效率哦
