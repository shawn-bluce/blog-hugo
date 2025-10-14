---
title: "防范 CSRF"
slug: "csrf-simple"
date: "2020-11-18T13:28:00+0000"
lastmod: "2025-01-16T07:38:12+0000"
draft: false
tags:
  - "Security"
  - "CSRF"
visibility: "public"
---
# 0X00 前言

![](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/csrf.png)

假设有这么一个银行（肯定要假设，如果真有这么蠢的话这银行也早就倒闭了）的网上银行站点，他提供了一套的 API，通过`GET https://api.xxbank.com/transfer?amount=3000&to=shawn`可以向 shawn 转账 3000 块。虽然用 GET 来修改数据是挺蠢的但是好像也没什么大问题是吧。

好的，现在你正在这个银行的网站上沉迷于数自己余额的零，这时候我给你发来了一个链接。你开开心心的看完了发给你的页面，然后关掉了它，回过头来发现自己账户少了 3000 块钱！！！怎么回事呢？

其实是我发给你的链接有“毒”，页面里所有的图都是`<img src="https://xxx.xxx.xxx/xxx.jpg"/>`，但是有一张图裂了你没发现，裂了的那张图是`<img src="https://api.xxbank.com/transfer?amount=3000&to=shawn" />`。这里有两个问题需要注意，第一个就是这个 url 并不是图片，所以图片必然会裂掉；第二个就严重了：因为是 `img` 标签，所以浏览器会去 GET 回来，这一 GET 没拿到图不要紧，却发起了我预谋的转账请求，你的钱就没得喽。

当然如果你多刷新几次，每次刷新就是 3000 块钱，嘿嘿嘿

# 0X01 什么是 CSRF

故事讲完了，那么到底什么是`CSRF`呢？

> 跨站请求伪造（英语：Cross-site request forgery），也被称为 one-click attack 或者 session riding，通常缩写为 CSRF 或者 XSRF， 是一种挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。跟跨网站脚本（XSS）相比，XSS 利用的是用户对指定网站的信任，CSRF 利用的是网站对用户网页浏览器的信任。 「[维基百科](<https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0>)」

说起 CSRF，不知道大家有没有听过一种“POST 请求比 GET 请求更安全”的说法，也许这种说法的来源就与 CSRF 相关，具体的我们后面会说到。

# 0X02 如何防范 CSRF

## 使用 Referer 字段

HTTP 协议中有一个 Referer 字段，是用来标记“发起这个请求的来源地址”，这样我们在后端做一个校验`if http_request.head.referer == 'https://www.xxbank.com/'`就可以了。这种做法的成本极低，只需要在后端新增一个校验，但是存在一个问题：不是非常靠谱。

为什么不是很靠谱呢？首先，这个字段并不是必填的，我们不能保证用户的牛鬼蛇神浏览器会传这个字段；其次，即使牛鬼蛇神传了这个值，也不能保证他是正确的。所以从这个角度上来说，虽然实现成本很低但是效果也并不太好。

## 使用 token 回传

首先我们要知道不要用 GET 去修改数据，不要用 GET 去修改数据，不要用 GET 去修改数据，然后再讨论接下来的问题。还是用转账这个 API，我们假设现在 API 是`POST https://api.xxbank.com/transfer/`，数据是`{"amount": 3000, "to": "shawn"}`。你可能会说“不对呀，还是可以在其他页面里用 Ajax 发送 POST 请求呀”，先别急，慢慢看。

我们要修改“转账”这个功能，本来只接受`amount/to`这两个参数的 API 现在接受第三个参数：`csrf_token`。那么这个`csrf_token`的值应该是什么呢？这个值又服务器生成，每次客户访问转账页面的时候带着返回个客户，然后客户表面上看起来这里有两个框框，填写金额和收款人，但实际上还隐藏了第三个框框，直面的值是自动填写的 `csrf_token`。现在用户点击“转账”后，发送的正确请求数据应该是`{"amount": 3000, "to": "shawn", "csrf_token": "2336666"}`了。

现在如果还有个小黑客用 Ajax 的方法发送请求，就不行了，因为这时候他只能填写`amount/to`这两个参数，第三个`csrf_token`是不知道的，所以防御成功。

## 双重 cookie

双重 cookie 并不是说有两个 cookie，而是说一个 cookie 要用到两次。当用户登录银行系统后，系统会发给他一个 cookie，这个完全没有问题对吧。但是采用了双重 cookie 验证的站点会在 cookie 里加入一个`csrf_tcookie=xxx`（当然你随便叫什么都可以）的值，接下来你在银行站点内发送的每一个请求都要取出这个`csrf_cookie`，拼在后面。

在不启用这个防护策略的时候，`POST https://api.xxbank.com/transfer/`就可以将钱转走，现在启用了这个策略，就必须得`POST https://api.xxbank.com/transfer/?csrf_cookie=xxxxx`才可以。然而由于这个值是放在 cookie 里的，所以只有该站点的请求才可以取到，其他站点的是不能取到 cookie 的。如此一来，csrf 的攻击也就失效了。（当然，这个的安全性是没有回传 csrftoken 这么强的）

# 0X03 参考

[美团技术团队前端安全系列（二）：如何防止CSRF攻击？](<https://tech.meituan.com/2018/10/11/fe-security-csrf.html>)
[维基百科：CSRF](<https://zh.wikipedia.org/zh/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0>)
