---
layout: post
title: "Object-C高级编程学习笔记(二)"
date: 2013-12-22 20:48:19 +0800
comments: true
categories: 
---

#Blocks

##What is blocks

###Block语法

Blocks是C语言的扩充功能：带有自动变量（局部变量）的匿名函数。

`^` `返回值类型` `参数列表` `表达式`

	^int (int count) {return count + 1;}

Block语法可以省略几个项目。首先是返回值类型：
 
 	^(int count) {return count + 1;}
 
其次，如果不使用参数，参数列表可以省略：

	^{ print(“Block\n”);}
	
使用Block语法讲Block赋值为Block类型变量：

	int(^blk) (int) = ^(int count) {return count + 1;}
	int(^blk1) (int) = blk;
	
使用typedef定义：

	typedef int (^blk_t) (int);
	blk_t blk = ^(int count) {return count + 1;}
	int nResult = blk(10);
	
Block类型变量可以像C语言中其他类型变量一样使用。

###截获自动变量值

Blocks中，Block表达式截获所使用的自动变量的值。因为Block表达式保存了自动变量的值，所以在执行Block语法后，即使改写Block中使用的自动变量的值也不会影响Block执行时自动变量的值。

	int a = 10;
	int b = 10;
	int (^blk) (void) = ^(return a + b;);
	b = 2;
	blk();
	
执行结果为：`20`

###__block说明符

若想在Block语法的表达式中将值赋给Block语法外的自动变量，需要在自动变量附加__block说明符，该变量成为__block变量。
	
	__block int val = 0;
	void(^blk) (void) = ^{val = 1;};
	blk();
	printf("%d",val);
	
执行结果为： `1`

赋值给截获的自动变量会产生编译错误：

	id array = [[NSMutableArray alloc] init];
	void (^blk) {array = [[NSMutableArray alloc] init]};

会出现编译错误。

另外，在使用C语言数组时，必须小心使用起指针。

	const char text[] = "Hello";
	void (^blk) (void) = ^{
		printf("%c\n",text[2]);
	};

在现在的Block中，截获自动变量的方法并没有实现对C语言数组的截获，因此会编译出错。可以使用指针解决：

	const char *text = "Hello";
	void (^blk) (void) = ^{
		printf("%c\n",text[2]);
	};	

##Blocks的实现

// TODO:需要一些时间理解