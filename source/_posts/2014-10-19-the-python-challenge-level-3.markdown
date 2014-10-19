---
layout: post
title: "The Python Challenge–Level 3"
date: 2014-10-19 10:35:11 +0800
comments: true
categories: 
---

Level3地址：[http://www.pythonchallenge.com/pc/def/equality.html](http://www.pythonchallenge.com/pc/def/equality.html)

## 分析

提示文字：

	One small letter, surrounded by EXACTLY three big bodyguards on each of its sides.
	
根据上一次的经验，查看网页的源码，标题为`re`，看来和正则表达式有关系了。

网页源码里仍然有一段被注释的字符串，看来就要在这一串中找出两边大写中间小写字母的内容了。

## 解决

	text = ‘’’
	       …
	       ‘’’
	import re
	pattern = re.compile(r'[A-Z]{3}([a-z])[A-Z]{3}')
	result = re.findall(pattern,text)
	print(‘’.join(result))
	
发现打印出来很多结果，哪里出了问题？

再仔细看提示语：`EXACTLY`，必须是有且只有三个大写字母。

修改正则表达式：
	
	pattern = re.compile(r'[^A-Z][A-Z]{3}([a-z])[A-Z]{3}[^A-Z]')
	
输出：

	linkedlist
	
## 其他解决方法

[http://www.pythonchallenge.com/pcc/def/linkedlist.php](http://www.pythonchallenge.com/pcc/def/linkedlist.php)

