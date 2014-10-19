---
layout: post
title: "The Python Challenge–Level 2"
date: 2014-10-19 09:15:54 +0800
comments: true
categories: 
---

## 分析

Level2的地址：[http://www.pythonchallenge.com/pc/def/ocr.html](http://www.pythonchallenge.com/pc/def/ocr.html)

文字提示`MAYBE they are in the page source`

查看网页的源码，发现注释的部分：
	
	find rare characters in the mess below:
	
需要从那段内容中找出稀少的字符。

## 解决

	text = ‘’’
	      …
	       ‘’’
	result = {}
	for x in text:
   	 	result[x] = result.get(x,0) + 1
	print(result)

输出：

	{'(': 6154, ')': 6186, '*': 6034, '+': 6066, 'l': 1, 'i': 1, ' ': 4880, '!': 6079, '#': 6115, '$': 6046, '%': 6104, '&': 6043, 'e': 1, 'y': 1, '{': 6046, '}': 6105, 'q': 1, 't': 1, 'u': 1, 'a': 1, '\n': 1221, '@': 6157, '[': 6108, ']': 6152, '^': 6030, '_': 6112}
	
发现最少的都是个数为1的字母，这样稍加改变：

	result = []
	for x in text:
    	if x.isalpha():
        	result.append(x)
	print(''.join(result))
	
输出：
	
	equality

## 其他的解决方法

按照惯例，查看解决方法：[http://www.pythonchallenge.com/pcc/def/ocr.html](http://www.pythonchallenge.com/pcc/def/ocr.html)

