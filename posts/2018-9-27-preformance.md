---
title: 前端性能分析总结（一）
date: Invalid date
tags: [blog]
published: true
hideInList: false
feature: 
---


这段时间忙到根本无暇写博客，刚有时间写了，又想着写这么一个大的题目，人生果然每天充满挑战。

## performance API

说说起因吧，有一天老大说

> “你在自己的模块里把性能分析加一下”

于是我看了下公司内部的某个性能分析库和平台，以及它的文档。其中有一张图引人瞩目

![](/images/get-file-event.png)

这张图从一种纬度来反应了用户从输入 url 到页面加载完成发生的各种事件。我们来详细看看，这些事件都代表什么意思。

### 事件周期

* navigationStart

    请求开始，用户在浏览器中输入了网址，并且点击了回车。

* redirectStart

    第一个 HTTP 请求发生重定向的时间，必须为同域名的跳转，否则值为 0。

* redirectEnd

    最后一个 HTTP 请求发生重定向的时间，必须为同域名的跳转，否则值为 0。

* unloadStart

    前一个网页 unload 的时间，必须与当前页面同域，否则为 0。

* unloadEnd

    前一个网页 unload 事件执行完毕的时间。

* fetchStart

    浏览器准备好发送 Http 请求文档，这发生在检查本地缓存之前。

* domainLookupStart

    DNS 域名查询开始的时间，如果使用了本地缓存（即无 DNS 查询）或持久连接，则与 fetchStart 值相等

* domainLookupEnd

    DNS 域名查询完成的时间，如果使用了本地缓存（即无 DNS 查询）或持久连接，则与 fetchStart 值相等

* connnetStart

    HTTP（TCP） 开始建立连接的时间，如果是持久连接，则与 fetchStart 值相等
    注意如果在传输层发生了错误且重新建立连接，则这里显示的是新建立的连接开始的时间

* secureConnectionStart

    HTTPS 连接开始的时间，如果不是安全连接，则值为 0

* connectEnd

    HTTP（TCP） 完成建立连接的时间（完成握手），如果是持久连接，则与 fetchStart 值相等
    注意如果在传输层发生了错误且重新建立连接，则这里显示的是新建立的连接完成的时间
    注意这里握手结束，包括安全连接建立完成、SOCKS 授权通过

* requestStart

    Http 开始真正请求文档的时间，包括从本地读取缓存。

* responseStart

    TTFB 开始接收响应的时间，包括从本地读取缓存。

* responseEnd

    Http 响应全部接收完成，包括从本地读取缓存。

* domLoading

    开始解析渲染 DOM 树

* domInteractive

    完成解析 DOM 树的时间

* domContentLoaded

    网页内资源加载开始的时间，DOM 树开始渲染，DOMContentLoaded事件完成之后的时刻，也是javascript类库中DOM ready事件触发的时刻。这里可以理解为 渲染树构建完成。

* domComplete

    所有资源已下载完毕，加载转环已停止旋转

* loadEventStart

    load 回调函数开始执行的时间

* loadEventEnd

    load 事件执行完毕

有了事件发生的时间，那我们如何利用这些事件？

> performance API

我们可以利用现代浏览器的 performance API 来进行事件的记录，API 里记录了事件发生的时间戳，我们可以因此完成性能统计。其实大多的性能分析库，就是利用这个 performance API 做到的。

### 相关 API

* performance.timing

记录了各个事件的时间点

* performance.getEntries()

包含了页面中所有的 HTTP 请求

* performance.now()

可以精确计算程序执行时间，不像 Date now 可能会受系统程序的影响而变得不精准，performance.now() 输出的是相对于 performance.timing.navigationStart(页面初始化) 的时间

* performance.mark()

可以通过传入 name ，便捷的批量标记事件发生点

* performance.memory

usedJSHeapSize 表示所有被使用的js堆栈内存；totalJSHeapSize 表示当前js堆栈内存总大小，这表示 usedJSHeapSize 不能大于 totalJSHeapSize，如果大于，有可能出现了内存泄漏。

* performance.navigation

    * performance.navigation.type

        * 0：网页通过点击链接、地址栏输入、表单提交、脚本操作等方式加载

        * 1：网页通过“重新加载”按钮或者location.reload()方法加载

        * 2：网页通过“前进”或“后退”按钮加载

        * 255：任何其他来源的加载

    * performance.navigation.redirectCount

        表示当前网页经过了多少次重定向跳转

> 只有大量的模仿之后，你才有能力去创造。





