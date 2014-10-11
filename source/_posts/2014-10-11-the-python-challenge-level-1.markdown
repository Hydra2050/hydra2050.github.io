---
layout: post
title: "The Python Challenge--Level 1"
date: 2014-10-11 21:46:08 +0800
comments: true
categories: 
---

Level 1的地址：[http://www.pythonchallenge.com/pc/def/map.html](http://www.pythonchallenge.com/pc/def/map.html)

## 分析

网页的标题为`What about making trans?`，然后再看图片：
	
	K -> M
	O -> Q
	E -> G
	
应该是一个类似凯撒密码的字母移位，每个字母向后移动两位，而要处理的就是下面那段文字。

##解决

代码如下：

	'''
	Level 1
	'''
	text = '''
	g fmnc wms bgblr rpylqjyrc gr zw fylb. rfyrq ufyr amknsrcpq ypc dmp. bmgle gr 	gl zw fylb gq glcddgagclr ylb rfyr'q ufw rfgq rcvr gq qm jmle. sqgle 	qrpgle.kyicrpylq() gq pcamkkclbcb. lmu ynnjw ml rfc spj.
	'''
	trans = ''
	for x in text:
    	trans += chr((ord(x) - ord('a') + 2)%26 + ord('a')) if x.isalpha() else x
	print(trans)

输出结果：

	i hope you didnt translate it by hand. thats what computers are for. doing it in
	by hand is inefficient and that's why this text is so long. using string.maketrans() 
	is recommended. now apply on the url.
	
根据转化后的信息，处理url中的map进行同样的转换，获得新的url：[http://www.pythonchallenge.com/pc/def/ocr.html](http://www.pythonchallenge.com/pc/def/ocr.html)，顺利进入下一关。

## str.maketrans()

等等，仔细看转换后的信息，推荐使用string.maketrans()方法。

查找[python3在线文档](https://docs.python.org/3/)，发现在python3.4后有所变化。

代码如下：

	''
	use str.maketrans()
	'''

	import string

	text = '''
	g fmnc wms bgblr rpylqjyrc gr zw fylb. rfyrq ufyr amknsrcpq ypc dmp. bmgle gr 	gl zw fylb gq glcddgagclr ylb rfyr'q ufw rfgq rcvr gq qm jmle. sqgle 	qrpgle.kyicrpylq() gq pcamkkclbcb. lmu ynnjw ml rfc spj.
	'''

	dlist = string.ascii_lowercase
	table = str.maketrans(dlist,dlist[2:] + dlist[:2])
	print(text.translate(table))
	
首先，使用str.maketrans()方法生成一个映射表，从[A－Z]映射为[C-A]，打印`table`：

	{97: 99, 98: 100, 99: 101, 100: 102, 101: 103, 102: 104, 103: 105, 104: 106, 105: 107, 106: 108, 107: 109, 108: 110, 109: 111, 110: 112, 111: 113, 112: 114, 113: 115, 114: 116, 115: 117, 116: 118, 117: 119, 118: 120, 119: 121, 120: 122, 121: 97, 122: 98}
	
然后，调用translate()方法，将映射表作为参数。最终获取想要的结果。

`str.maketrans()`的第一个和第二个参数长度必须相同，转换的时候一一对应；如果有第三个参数，将会转换为`None`。

	mm = 'abcdefg'
	tb1 = str.maketrans('abcd','1234')
	tb2 = str.maketrans('abcd','1234','ef')
	print(mm.translate(tb1))
	print(mm.translate(tb2))
	
输出结果：

	1234efg
	1234g
	
## 其他的解决方法

进入下一个Level的页面，有一条tips，将当前url中的`pc`改为`pcc`就可以查看前一个Level的解决方法了。

[http://www.pythonchallenge.com/pcc/def/ocr.html](http://www.pythonchallenge.com/pcc/def/ocr.html)


参考资料：

[Python documentation](https://docs.python.org/3/)
