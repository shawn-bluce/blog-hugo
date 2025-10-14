---
title: "写给 git 新手的 6 个小技巧"
slug: "git-tips"
date: "2023-07-31T14:18:00+0000"
lastmod: "2025-03-04T08:52:59+0000"
draft: false
tags:
  - "Git"
visibility: "public"
---
# 0X00 header

首先，git 是没有 header 这个命令的:)

平时经常会用到一些 git 的用法，也有遇到过别人来问怎么实现某某操作，但是都太零散了并不成体系，所以这里就简单整理一下不做分类了。希望对不小心通过 Google 搜到该文章的你有所帮助（不会的有人能搜到吧🤣）

# 0X01 pull 与 pull --rebase

首先是搞清楚 `git pull` 和 `git pull --rebase` 的区别，我们学 git 第一天肯定就学了 `git push` 是推代码，`git pull` 是拉代码，那么这个 `git pull --rebase` 又是个什么东西呢？

`git pull` 本质上是 `git fetch` 加上 `git merge` 的功能，也就是说把代码拉到本地来再和本地代码进行一次 merge。这就意味着，如果出现冲突的话 git 会创建一个 Merge commit 来解决这个问题。

`git pull --rebase` 则是 `git fetch` 和 `git rebase` 的组合。使用的时候会先将本地比远端多出来的 commit 暂存起来，等将本地代码与远端代码同步后再将暂存的 commit 逐个应用。这样一来提交历史会更加整洁（因为没有额外的 Merge），但是需要注意的是使用 `rebase` 可能会造成 commit 历史发生变化。

> 吐槽一句，这个 rebase 被中文翻译成「变基」虽然确实合理，但真的好怪啊🤔

# 0X02 rebase -i

相信各位在践行 git-flow 工作流的时候一定遇到过这种情况：在自己的开发分支仓库中提交了代码，测试过后发现有问题，就提交 fix，再发现有问题就再 fix，最后合并到主仓库的时候发现有十几个 commit。这种情况呢其实直接将这十几个 commit 直接合并过去也不是不行，但是就会显得历史记录很混乱了，这种时候我就比较喜欢将几个修改同一处代码/同一个功能的连续的 commit 合并成一个。

这里可以看到这个我测使用的仓库在 create a 之后一直在 update a，那我现在想将他们合并就可以用 `git rebase -i fba6a4d6b6364e9d3e485db366aeedb86c776904` 命令将 `fba6a4d6b6364e9d3e485db366aeedb86c776904` 之前的所有 commit 合并起来（也就是所有的 update）。

![SCR-20230731-sxyt](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2023/07/31/scr20230731sxyt.png)

使用这个命令之后进到这里，把不需要单独保留的 commit 的 pick 全部改成 `s` 也就是 squash，再保存退出。具体的 pick 是什么含义、s 是什么含义，在这个页面的下面有官方的文档解释，可以自行阅读。

![SCR-20230731-syle](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2023/07/31/scr20230731syle.png)

下一个界面就是编辑新的 commit message，我这里只需要将多余的删掉就可以了。

![SCR-20230731-synt](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2023/07/31/scr20230731synt.png)

最后可以看到，所有的 update 就都合并成同一个 commit 了。

![SCR-20230731-sysy](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2023/07/31/scr20230731sysy.png)

# 0X03 cherry-pick

cherry-pick 的用法就比较简单了。我们假设有一个仓库，它有一个主分支 master，还有两个额外给客户定制的版本分别是 branch_A 和 branch_B，他们都是从 master fork 出来的（这种场景还是非常合理的哈）。

这时候主线分支进了一个重要的 hotfix 是修复某安全漏洞的，那当然也要将这个改动应用到那两个定制分支上去。显然是不可能记住改动然后在两个分支上单独去改的。这时候就可以用到 `git cherry-pick` 了，首先要切到 master 分支，然后将修复的 commit id 复制下来，假设它是 `123123abcabc`，这时候再切到 branch_A 执行 `git cherry-pick 123123abcabc`，最后将弹出来的编辑器内容检查一下保存就可以了。这样就将这个 commit 的所有改动直接迁移到 branch_A 上了，branch_B 也是同理。

> 需要注意的两点：如果有多个 commit 需要迁移，记得按照提交时间从旧到新的顺序逐一操作cherry-pick 的时候需要注意可能会出现的冲突

# 0X04 commit --amend

各位一定都有手抖的时候是吧，比如刚刚提交的 commit message 发现有问题需要修改。这时候就直接使用 `git commit --amend` 命令，就可以直接修改最近一条 commit message 了，就像你刚刚用了 `git commit` 一样。

# 0X05 reset --soft HEAD~

上面的命令是可以回退到编辑 commit message 的时候，而 `git reset --soft HEAD~` 命令则是直接撤销掉最近一条 commit。在你的代码仓库中执行这条命令后 git 会将你最近的一条 commit 直接撤回，并将改动置为 `stage` 状态随时可以再次提交。

# 0X06 checkout PATH

最后一个是 `checkout` 在切换分支之外的功能，最常见的是例如 `git checkout .` 用法。这个用法可以将当前目录及其递归子目录中的所有没有在暂存区（没有被 `git add` 过的）的改动全部清空。

比如你在 dev 分支做了一些改动，用来调试一些问题，最终肯定是不想提交代码的，这时候就可以回到项目的根目录用 `git checkout .` 来清空这些不再需要的改动。
