---
title: 移动端调试总结
date: Invalid date
tags: [blog]
published: true
hideInList: false
feature: 
---


这半年多以来，一直在做移动端的网页开发，在移动端网页中，一大痛点就是调试代码，如何快速的定位问题，如何在特定的手机和浏览器中调试代码。

在这里总结一下，下面的调试方法会冥冥之中有一个顺序，也可以当作我们选择调试时的一个顺序。以下方法的具体操作细节我不再赘述，网上有很多。

## Eruda

> Eruda 是一个专为手机网页前端设计的调试面板，类似 DevTools 的迷你版，其主要功能包括：捕获 console 日志、检查元素状态、捕获XHR请求、显示本地存储和 Cookie 信息等等

其实我们比较熟知的调试面板工具是 [Vconsole](https://github.com/Tencent/vConsole) ，但从功能上，我们可以明显看出，Eruda 在 Vconsole 之上。

使用方法和 Vconsole 大同小异：

* 在页面引入 js

* 初始化方法

这里我们注意一下：

The JavaScript file size is quite huge(about 100kb gzipped) and therefore not suitable to include in mobile pages. It's recommended to make sure eruda is loaded only when eruda is set to true on url(http://example.com/?eruda=true), for example:

```javascript
;(function () {
    var src = 'node_modules/eruda/eruda.min.js';
    if (!/eruda=true/.test(window.location) && localStorage.getItem('active-eruda') != 'true') return;
    document.write('<scr' + 'ipt src="' + src + '"></scr' + 'ipt>');
    document.write('<scr' + 'ipt>eruda.init();</scr' + 'ipt>');
})();
```
[库地址](https://github.com/liriliri/eruda)

## charles

> Charles is an HTTP proxy / HTTP monitor / Reverse Proxy that enables a developer to view all of the HTTP and SSL / HTTPS traffic between their machine and the Internet

charles 是一个抓包工具，这里之所以可以用来 debug ，是因为 charles 可以：

* 修改网络协议

* 替换请求文件

当我们需要用本地的 js 文件来代替线上 js 文件进行调试时，charles 就帮了我们大忙，也弥补了 Eruda 无法对 js 进行太多操作的缺点。

charles 进行文件替换的原理很简单，就是在本地开了一个服务，然后起到了可以替换请求文件的中间件的作用。

使用方法：

* 开启 charles 软件

* 手机与电脑在 wifi 下

* 手机设置代理

这里要注意的一点是，如果需要监听手机的 https 请求，需要给手机安装 charles 证书。

## whistle

> whistle is a cross-platform web debugging tool based on Node.js

whistle基本上覆盖了所有抓包调试代理可以实现的功能，它的横向对比工具还有 [spy-debugger](https://github.com/wuchangming/spy-debugger)。

所以既然 whistle 可以代替 Eruda 和 charles（太重） ，那我们就没有必要使用它们了。

whistle 的功能很强大：

* Weinre

* Log

    Log 主要是为了把控制台打印的信息展示出来，规则配置也很简单

    > xx.yy.com log://xx
    
* proxy

    代理的方法也很简单，在 whistle rules 里面配置需要代理的网址，然后用规则来转到线下

    > pattern http://host:port/xxx

## Safari 与 iphone

其实，上面所讲的抓包调试代理有一个很大的缺点，那就是无法 debug js。在 iphone 机型下，为我们提供了解决方案。

使用方法：

* Mac Safari

* iphone Safari

* 数据线

## Chrome 与 Android

当然，在 android 机型下，也有相对应的解决方案。

使用方法：

* PC Chrome

* Android Chrome

* 数据线

* 翻墙

在使用这种调试方法的时候，可能会出现一些问题

* inspect 后页面显示空白

    解决方案：原因是被墙了，此服务需要连接谷歌的服务器，解决方法是翻墙，一般只需第一次 inspect 时翻，之后会缓存在本地。

* inspect 页面显示 HTTP/1.1 404 Not Found

    解决方案：原因是因为远程的浏览器（手机）版本比客户端（电脑）要新，需要在 chrome://inspect/#devices 下，使用 inspect fallback 进行监听。

> `Safari 与 iphone` `Chrome 与 Android` 不能在手机的其他浏览器和 App webview 中使用，这可能会让我们在调试一些特定浏览器问题上没有解决途径，这是就需要搭配一些其他手段。比如 本地代理 + try catch

## 总结

* `Eruda` `charles` `whistle` 在解决问题时用到的较多

* `Safari 与 iphone` `Chrome 与 Android` 在发现问题时用到的较多

* 如果问题发生在特定浏览器上，可以搭配本地代理( whistle 等) + try catch 定位 bug

> 这个世界就是不讲道理