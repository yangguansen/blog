---
title: HTTP缓存机制
date: 2018-03-31 12:15:45
tags:
	- 总结
---
深入理解HTTP缓存
<!--more-->

> 前言

在网上看了无计其数关于HTTP缓存的文章，在这里记录下自己的理解和总结，尽量使用最简单的流程来缕清缓存机制的从头到尾的过程。

> 疑问

作为前端开发者，对于浏览器调试界面的network肯定都有所了解，以一个图片资源为例，我们刷新页面，network中此图片资源时而返回status 200,时而返回200（from disk cache）,时而返回304 （not modify），我们都知道是缓存导致状态码的不同，但是到底是什么情况下才会返回对应的状态码呢，傻傻分不清楚。

> 解析

假如我们访问这个图片资源，服务器会返回资源文件和关于这个资源的响应信息如下：

```
(Status-Line)      HTTP/1.1 200 OK
Date                Sat, 31 Mar 2018 03:42:12 GMT
Server              Apache/2.2.11 (Unix)PHP/5.2.9
Last-Modified       Thu, 26 Nov 2009 13:50:19 GMT
Expires             Wed, 23 Dec 2018 06:04:03 GMT
Etag                “8fb8b-14-4794674acdcc0″
Cache-Control       max-age=1000
Accept-Ranges       bytes
Content-Length      20
Keep-Alive          timeout=5, max=100
Connection          Keep-Alive
Content-Type        text/html
```

以上信息中，关于缓存的key有 `Last-Modified` `Etag` `Expires` `Cache-Control`,

- Last-Modified: 上次资源修改时间

- Etag: 资源的标识符,每次修改资源都会对应生成一个标识符来表示此版本的资源,可以理解成资源版本号

- Expires: 资源过期时间，返回的是服务器时间

- Cache-Control: 资源缓存有效期，以秒为单位


**HTTP缓存机制分为强缓存和协商缓存：**

· Expires和Cache-Control是触发强缓存的key

· Last-Modified和Etag是触发协商缓存的key

**强缓存优先级高于协商缓存，也就是说，会先看强缓存，强缓存生效，就直接使用强缓存，强缓存过期，再去使用协商缓存。**


那么当浏览器加载资源时，缓存机制的处理流程如下：

1、查看Cache-Control和Expires过期没，如果这两项都没过期，就直接使用浏览器目录chche文件下的缓存文件,返回状态码200（from disk cache），这一步，没有访问服务器。

2、假如Cache-Control和Expires都过期了，HTTP请求中就携带Last-Modified和Etag（协商缓存）去访问服务器资源，服务器会根据HTTP带来的信息和当前服务器关于此资源的最新的Last-Modified和Etag进行对比，对比结果有两种：

- 如果HTTP中的Last-Modified和Etag信息和服务器中的Last-Modified和Etag信息一致，就说明没有修改，返回状态码304(Not modify)。

- 对比结果不一致，返回状态码200,并返回新的资源文件。

以上便是自己对缓存机制的理解，如有写的不对的地方，还望指正互相交流。


（完）