---
title: "git 中的 bare repository"
slug: "git-base-repository"
date: "2018-08-19T14:24:00+0000"
lastmod: "2025-01-16T08:53:14+0000"
draft: false
tags:
  - "Git"
visibility: "public"
---
# 0X00 遇到了什么

我们使用git，绝大多数情况下都是大于等于一个人进行代码编辑，然后将自己的改动提交到`github/gitlab/gogs`等仓库，然后再通过`pull request/merge request`的方式进行代码合并。所以我们一般都是先从`github`上创建一个新的项目，然后按照向导在自己的本地`git clone`下来一个空项目，再提交代码上去；或者`fork & clone`的流程。

以前从来没有去想过`github`上的仓库是不是和我本地的相同，以至于今天第一次搭建自己的git服务时遇到了问题。我从服务器上一顿操作猛如虎`mkdir xxx; cd xxx; git init; touch README.md; git add .; git commit -m "init the repository"`，结果不小心成了二百五。因为这个仓库根本不能clone到我本地，经过一番搜索发现了git中仓库之间的关系没有这么简单。

# 0X01 git中一般的repository

git中一般的repository通常有两个来源：`git clone`或者在`git init`。这种仓库通常是我们用来正常工作的仓库，我们的代码都在这里面。我们可以在仓库里尽情使用`git add/rm/status/log`等常见操作。也可以将代码push到`origin/upstream`等上游仓库，这类仓库是最常见的了。

仓库中不仅存有我们正在使用的代码，有两个文件`.gitignore`和`.git`。其中`.gitignore`自然不必多说，是用来判断哪些文件需要或不需要提交到repository中的；另一个`.git`就是git实现版本控制的重要文件了。我们对代码的改动、不同的分支、commit的变化都是在这个目录中存储的。

> 不建议直接动手修改`.git`目录中的任何文件，有可能会造成奇怪的问题而无法解决。

# 0X02 git中的bare repository

其实git中还有一种`bare repository`，这种仓库中文可以称之为`裸仓库`。这种仓库中没有我们正常使用的代码，也没有`.gitignore`和`.git`文件。不过可以把这种仓库理解成将`.git`目录下所有文件都拿到`bare repository`仓库的根目录了。`bare repository`存储不同分支与各个commit等与版本控制相关的数据，但是不会保存整整一份代码。

但是这种仓库可以使用`git clone`来clone仓库到本地，所以通常被当作共享仓库。例如你的团队没有在使用github或是gitlab等工具，那就可以在一台服务器上创建一个`bare repository`，然后大家均从此仓库clone代码，然后再一起提交至此以实现git的工作流。

要生成一个空的`bare repository`很简单，只需要在一个空目录中`git init --bare`就可以了，生成好之后就可以从该仓库`clone`代码了。

# 0X03 部署一个服务器上的repository

到这里部署一台合作用的服务器上的repository就变得容易多了。首先创建一个git使用的用户，再使用git用户创建对应的裸仓库，再从客户机clone到本地，接下来就可以正常使用`commit/push/pull`等操作了。

  1. 在服务器上创建一个git用户`user add git`
  2. 然后为其创建home目录`mkdir /home/git`
  3. 再修改所属人为git`chown git.git /home/git`
  4. 切换成git用户并返回home目录`su git; cd`
  5. 在目录下创建一个空目录`mkdir .ssh`
  6. 在`.ssh`，目录中创建一个`authorized_keys`的文件，在里面填入自己的ssh公钥
  7. 在本地已经可以clone下来啦`git clone ssh://git@xxx.xxx.xxx.xxx/xxx/xxx.git`
  8. 为了安全可以修改`git`用户的默认shell`usermod git -s /bin/git-shell`
