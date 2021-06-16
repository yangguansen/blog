---
title: Web安全头
date: 2021-06-16 09:50:10
tags:
    - 总结
---

本文列出了可用于保护网站的最重要的安全标头。使用它来了解基于 Web 的安全功能，了解如何在您的网站上实施它们，并作为您何时需要提醒的参考。
<!--more-->

> 目前应用在项目中的安全头有如下：
```
//  内容安全策略
Content-Security-Policy

//  同源才能通过iframe引入
res.header('X-Frame-Options', 'SAMEORIGIN');

//  只能被同源的网站引入资源
res.header('Cross-Origin-Resource-Policy', 'same-origin');

//  只能被同源网站通过open()打开
res.header( 'Cross-Origin-Opener-Policy', 'same-origin');

//  规定时间内，都应该通过HTTPS访问，并且包括子域
res.header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');

```
以上安全头可以兼容大部分浏览器，接下来针对每一个安全头进行细致讲解

> Content-Security-Policy

该header是配置在meta标签中，表示document加载内容的安全策略，可以有效的防止XSS攻击。比如他可以限制禁止加载某些外部资源，
限制禁止执行script脚本。
可以限制的资源类型如下：
-   script-src：外部脚本
-   style-src：样式表
-   img-src：图像
-   media-src：媒体文件（音频和视频）
- font-src：字体文件
- object-src：插件（比如 Flash）
- child-src：框架
- frame-ancestors：嵌入的外部资源（比如<iframe\>）
- connect-src：HTTP 连接（通过 XHR、WebSockets、EventSource等）
- worker-src：worker脚本
- manifest-src：manifest 文件

每个资源类型的配置项如下：

- 主机名：example.org，https://example.com:443
- 路径名：example.org/resources/js/
- 通配符：*.example.org，*://*.example.com:*（表示任意协议、任意子域名、任意端口）
- 协议名：https:、data:
- 关键字'self'：当前域名，需要加引号
- 关键字'none'：禁止加载任何外部资源，需要加引号

比如我的应用中配置如下：
```
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self' 'unsafe-inline'; font-src * data:; object-src 'none'; img-src * data: blob:; style-src 'self' 'unsafe-inline'">
```

> X-Frame-Options

在HTML中，我们可以通过`<iframe/>`标签引入第三方网址，使用iframe攻击的方式就可以是`点击劫持`。
使用`X-Frame-Options`可以设置自己的应用是否允许被外部引入。主要有以下配置：
```
X-Frame-Options: DENY   //  拒绝引入
X-Frame-Options: SAMEORIGIN   //  只限于同源引入
```

> Cross-Origin-Resource-Policy

该安全头是指静态资源是否允许被外部网站引入
配置如下：
```
Cross-Origin-Resource-Policy: same-origin   //  允许被符合同源策略（协议、域名、端口）的站点引入
Cross-Origin-Resource-Policy: same-site     //  允许被相同站点引入（只要二级域名相同就行）
Cross-Origin-Resource-Policy: cross-origin  //  允许被跨域引入
```

关于`same-site`和`same-origin`的区别，可以见这片文章[链接](https://web.dev/same-site-same-origin/)

> Cross-Origin-Opener-Policy

该安全头是指是否允许通过`window.open()`打开的窗口与自身窗口进行通信。
众所周知，window.open()可以打开一个新的窗口加载外部链接，攻击者可以使用postMessage在
两个站点之间进行通信。
配置如下：
```
Cross-Origin-Opener-Policy: same-origin     //  将文档与跨源文档窗口隔离。
Cross-Origin-Opener-Policy: same-origin-allow-popups    //  为弹出窗口打开时仍然可以保护文档不被引用，但允许它与自己的弹出窗口进行通信
Cross-Origin-Opener-Policy: unsafe-none     //  默认值，该文档可以通过跨域窗口打开并保留相互访问
```

> Strict-Transport-Security

简单来说，当我们访问一个配置了https的http链接，服务端会重定向到https链接。
改header通知浏览器它不应该使用 HTTP 加载站点，而是使用 HTTPS。设置后，浏览器将使用 HTTPS 而不是 HTTP 来访问域，而无需在标头中定义的持续时间内进行重定向。

只需要配置时间即可：
```
Strict-Transport-Security: max-age=31536000
```
服务器http到https的重定向的状态码为301
在设置的这个时间内，浏览器都会自动将http转为https，而不用服务器去重定向，返回的状态码为307。