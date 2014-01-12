---
layout: post
title: "HTTP的缓存机制"
date: 2014-01-12 13:56:24 +0800
comments: true
categories: 
---

##前言

HTTP的缓存机制，主要体现在HTTP协议头里面的几个字段——Expires、Cache-control、Last-Modified/If-Modified-Since、Etag/If-None-Match等。在移动开发中，了解HTTP的缓存对开发者来说已经必不可少。

![HTTP HEAD](/images/2014011216.png)

##Expires

HTTP头中的Expires字段，告诉浏览器在Expires显示的时间前，浏览器可以从缓存中读取，而不需要再次去请求。

##Cache-control

Cache-control与Expires一样，都是告诉浏览器有效期。不过它不只是设置过期时间，还可以设置很多选项：

如图：max-age＝600，表示600s内不需要重新请求。

##Last-Modified

`Last-Modified`：资源的最后修改时间。

如果本地缓存已经过期，即超过了`max-age`的缓存时间，缓存的HTTP头中存在Last-Modified，则向服务器发送带有If-Modified-Since字段的请求，后面带上`Last-Modified`所记录的最后修改时间。

服务器收到带有`If-Modified-Since`的HTTP请求，会将请求的资源修改时间与`If-Modified-Since`时间对比：如果这段时间没有被修改过，则返回304，告诉浏览器可以继续使用本地缓存；如果资源已经修改过了，则响应请求的资源数据，返回200 OK.

##Etag

`Etag`：服务器应答数据的时候，生成的当前资源的唯一标识。

如果本地缓存已经过期，即超过了`max-age`的缓存时间，缓存的HTTP头中存在`Etag`，则向服务器发送带有`If-None-Match`字段的请求，后面带上`Etag`所记录的当前资源的唯一标识。

服务器收到带有`If-None-Match`的HTTP请求，会将请求的资源唯一标识与`If-None-Match`的标签的标识对比：如果这段时间没有被修改过，则返回304，告诉浏览器可以继续使用本地缓存；如果资源已经修改过了，则响应请求的资源数据，返回200 OK。

##Etag与Last-Modified的区别？

HTTP1.1中Etag的出现主要是为了解决几个Last-Modified比较难解决的问题：

*  Last-Modified标注的最后修改只能精确到秒级，如果某些文件在1秒钟以内，被修改多次的话，它将不能准确标注文件的修改时间
*  如果某些文件会被定期生成，当有时内容并没有任何变化，但Last-Modified却改变了，导致文件没法使用缓存
*  有可能存在服务器没有准确获取文件修改时间，或者与代理服务器时间不一致等情形

`Etag`是服务器生成的对应资源的唯一标识符，能够更加准确的控制缓存。`Last-Modified`与`ETag`是可以一起使用的，服务器会优先验证`ETag`，一致的情况下，才会继续比对`Last-Modified`。

##跳转、刷新、强制刷新

浏览器缓存行为还有用户的行为有关:

用户操作      | Expires/Cache-control | Last-Moified/Etag
------------ | :-------------------: | :---------------:
地址栏回车     | 有效                  | 有效
页面链接跳转   | 有效                  | 有效  
新开窗口      | 有效                  | 有效
前进、后退     | 有效                  | 有效 
F5刷新        | 无效                  | 有效
Ctrl＋F5刷新  | 无效                  | 无效 

##总结

当本地存在缓存的时候，一次请求的流程：

![HTTP Cache](/images/HTTPCache.png)
