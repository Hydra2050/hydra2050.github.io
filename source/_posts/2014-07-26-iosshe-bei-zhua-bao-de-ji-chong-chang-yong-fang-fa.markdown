---
layout: post
title: "在Mac上iOS开发抓包的几种常用方法"
date: 2014-07-26 21:56:07 +0800
comments: true
categories: 
---

在iOS开发中，经常需要抓包来查看网络请求、应答的情况，例如坚持服务端应答数据、服务端应答速度、客户端发生请求、流量检测、安全性检测等等。不论代码内如何实现，抓包能够最好的证实一切，往往可以更快地定位问题，解决问题。下面总结几种常用的抓包方法。

## 使用charles抓包

在mac下，[charles](http://www.charlesproxy.com/)是一个非常好用的抓取http/https请求的工具。

限制条件：

* 需要mac与iOS设备连接相同的无线网络
* 只能抓取http/https的数据包 
* 无法抓取2G/3G/4G网络下得数据

［注］：更多关于charles的使用方法参照[http://www.charlesproxy.com/documentation/](http://www.charlesproxy.com/documentation/)

## 使用RVI（Remote Virtual Interface）+wireshark抓包

在iOS5以后，Apple为iOS引入了RVI（Remote Virtual Interface），只要设备通过USB连接到Mac，就可以虚拟出一个远程端口，这样就可以在Mac上使用Wireshark抓取这个远程虚拟端口的数据包了。

限制条件：

* 支持的设备系统iOS5以及之后

## 使用tcpdump命令抓包

限制条件：

* 需要iOS设备越狱

## 小结

以上是平时常用到得几种抓包方法，各有优缺点，可以根据需要选择最方便的方法。

## 参考资料

[http://www.charlesproxy.com/](http://www.charlesproxy.com/)

[http://www.charlesproxy.com/documentation/](http://www.charlesproxy.com/documentation/)

[http://www.wireshark.org/](http://www.wireshark.org/)