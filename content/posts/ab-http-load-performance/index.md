---
title: "使用 ab 和 http_load 进行简单的性能测试"
slug: "ab-http_load-performance"
date: "2020-11-04T15:31:00+0000"
lastmod: "2025-01-16T07:21:52+0000"
draft: false
tags:
  - "Performance"
  - "HTTP"
  - "Tools"
visibility: "public"
---
# 0X00 概述

我们经常想要简单测试一下某个页面的性能，或者 REST API 的性能，用 postman 这种程序虽然可以很快的模拟请求出来但是并不方便发送大量请求；自己写脚本虽然自定义程度很高，但是写起来总归还是有点麻烦，而且并不能很方便得输出我们需要的一些常用数据。那这种时候一般就需要找些专业的工具来做这种专业的事情了。下面两个就是平时比较常用的性能测试工具，可以针对某个/某些 url 做性能测试，并且输出相对完整的报告。

软件具体怎么装就自己想想办法吧，毕竟 Linux/macOS/windows 之间都不太一样，而且 Linux 上还有 apt/dnf/pacman 巴拉巴拉一堆包管理器，用法都不一样，没必要一个个找出来贴上。需要了解 API 性能的人肯定能很轻松装上这些工具了~

# 0X01 apache bench

最简单的使用方式：`ab -n 20 -c 2 https://example.org/`回车，就会执行一次简单的测试并输出报告。这串命令会用`GET`请求访问`https://example.com/`，并发为 2，总共发起 20 次请求。**需要注意，测试的时候域名最后已经加上`/`，因为测试是针对 url 的，而非 domain**

接着我们会得到一个这样的输出结果。其中第一部分是服务器的相关信息；第二部分是本次测试的相关信息；第三部分是连接相关的信息；第四部分是请求耗时的相关信息，我们只是简单看一下请求的响应速度，所以重点看第三和第四部分。

![](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/ab_result.png)

第三部分内容是连接耗时的数据，分别统计了连接、处理、等待的最小、平均、中位、最大值。

作为简单的测试结果，第四部分是最值得关注的，工具统计出了所有请求的耗时时间分布。我的这条结果显示 50% 的请求在 592ms 内完成；90% 的请求在 839ms 内完成，100% 的请求在 1069ms 内完成（也就是说最慢的一次请求用了 1069ms）。

简单的`GET`请求就到此为止了。如果需要携带 Header（这太常见了，起码登录过后得带个 token 才能真正测试），那就携带新的参数`-H `即可，类似于这个样子`ab -n 10 -c 2 -H 'token: xxxxxxx' https://example.org/`。如果需要携带多个 Header 参数，那就多接几个`-H`也没问题。

如果要发送 `POST` 请求，需要先把请求内容写在一个文件里，例如在`/homes/shawn/post_data`，然后通过`-p /home/shawn/post_data`就可以把文件内容通过 `POST` 请求发送出去了，比如这样` ab -n 2 -c 1 -p /home/shawn/post_data http://127.0.0.1:2333/`

# 0X02 http_load

`http_load`可以测试多个 url，不用像 `ab` 那样只能同时测单一 url；但是问题是不能使用`GET`以外的其他方法，所以用途也比较单一，用法也比较简单。

常用 4 个参数：

  1. -p --parallel 并发进程数
  2. -f --fetches 总访问数
  3. -r --rate 每秒访问数
  4. -s --seconds 访问总时间

首先要准备一个纯文本文件，类似这样`/home/shawn/urlist`的文件

```
    https://blog.just666.com/about/
    https://blog.just666.com/about/
    https://blog.just666.com/about/
    https://blog.just666.com/about/
    https://blog.just666.com/about/
    https://blog.just666.com/about/
    https://blog.just666.com/about/
    https://blog.just666.com/about/
    https://blog.just666.com/about/
    https://blog.just666.com/about/
    https://blog.just666.com/about/
    https://blog.just666.com/about/
    https://blog.just666.com/about/
    https://blog.just666.com/about/
    https://blog.just666.com/about/
    https://blog.just666.com/
    https://blog.just666.com/
    https://blog.just666.com/
    https://blog.just666.com/
    https://blog.just666.com/
    https://blog.just666.com/
    https://blog.just666.com/
    https://blog.just666.com/
    https://blog.just666.com/
    https://blog.just666.com/
    https://blog.just666.com/
    https://blog.just666.com/
    https://blog.just666.com/
    https://blog.just666.com/
    https://blog.just666.com/
```

保证每行一个 url，接下来就是`http_load -p 3 -f 10 /home/shawn/urllist`以 3 的并发量发起 10 次请求，或者`http_load -r 5 -s 10 xxx` 以每秒 5 次的频率请求 10 秒

会得到类似这样的输出结果

![](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/http_load.png)

总共发起了 19 次请求；并发为 1；大小是697449 btes；总共用了 10.0025 秒；平均每个连接是36707.8 bytes；平均每次请求 1.89 秒；平均响应是 20ms、最大 125、最小 8.2；

最后是各个状态码的数量，这里 19 个都是 200，一切正常；如果 50x，尤其是 502/503 这种就有可能是你频繁请求导致服务器不稳定了。

中间缺了一行`msecs/first-response`的解释？实话说我也不知道。manual 里没有这个解释，google 没找到官方的解释，二手资料全是复制自同一个人的，结果最初的那个老哥可能也没搞明白就没写。。。
