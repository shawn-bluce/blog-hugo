---
title: "git stash 命名 / git stash 用法「重置版」"
slug: "git-stash-remake"
date: "2022-06-15T14:04:00+0000"
lastmod: "2025-01-16T08:56:38+0000"
draft: false
tags:
  - "Git"
visibility: "public"
---
# 0X00 前言

本篇文章是这篇「[使用 git stash save 将暂存区命名](<__GHOST_URL__/2018/03/05/git-stash/>)」的重置版。因为根据 Google 的统计数据我得知某些问题的关键词搜索出来之后我的博客排行会比较靠前，所以把最容易被各位点击到的文章做了个重置计划，改进之前的一些不足，争取能够说的更清楚一些，也能节省各位一点点时间，希望能真正的帮助到从搜索引擎点进来的各位~

# 0X01 极度精炼的使用说明

可以使用 `git stash save "message"` 的方式为 stash 起来的变动命名，方便后面再次使用。

![git-stash-save](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/06/15/gitstashsave.png)

stash 起来过后可以使用 `git stash list` 来查看已经被 stash 的列表，这里可以看到已经有两条了，值得注意的是 stash 的 id 每次都在更新，最近 stash 的是 0，1 就是上次 stash 的，以此类推

![git-stash-list](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/2022/06/15/gitstashlist.png)

如果需要从新应用某个 stash 的改动，可以使用 `git stash apply STASH_ID` 的方式，例如使用 `git stash apply 1` 就可以重新应用 id 为 1 的这个 stash。如果要丢弃掉某个 stash 的话使用 `git stash drop STASH_ID` 就可以了
