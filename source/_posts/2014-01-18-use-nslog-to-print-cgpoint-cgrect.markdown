---
layout: post
title: "如何使用NSLog打印CGPoint、CGSize、CGRect"
date: 2014-01-18 22:02:46 +0800
comments: true
categories: 
---

平时调试程序的时候，经常会使用NSLog打印出来。当有CGPoint、CGSize、CGRect对象要打印时，一般要写成下面的方式：

	CGPoint point = CGPointMake(0, 20);
    NSLog(@"Point: x=%f, y=%f", point.x, point.y);
    
    CGSize size = CGSizeMake(200, 100);
    NSLog(@"Size: w=%f, h=%f", size.width, size.height);
   
	CGRect rect = CGRectMake(0, 0, 200, 100);
    NSLog(@"Rect: x=%f, y=%f, w=%f, h=%f", rect.origin.x, rect.origin.y, rect.size.width, rect.size.height);
    
今天看到一篇文章：[How to Use NSLog to Debug CGRect and CGPoint](http://iosdevelopertips.com/debugging/how-to-use-nslog-to-debug-cgrect-and-cgpoint.html)，才了解到有一些其他方便的方法。

	CGPoint point = CGPointMake(0, 20);
    NSLog(@"Point: %@", NSStringFromCGPoint(point));
    
    CGSize size = CGSizeMake(200, 100);
    NSLog(@"Size: %@", NSStringFromCGSize(size));
    
    CGRect rect = CGRectMake(0, 0, 200, 100);
    NSLog(@"Rect: %@", NSStringFromCGRect(rect));
   
打印结果：

	Point: {0, 20}
	Size: {200, 100}
    Rect: \{\{0, 0\}, \{200, 100\}\}


