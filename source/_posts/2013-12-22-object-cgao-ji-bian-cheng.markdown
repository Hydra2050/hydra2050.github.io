---
layout: post
title: "Object-C高级编程学习笔记(一)"
date: 2013-12-22 14:28:55 +0800
comments: true
categories: 
---

书名：《Object-C高级编程 IOS与OS X多线程和内存管理》

##ARC

在Object-C中采用Automatic Reference Counting(ARC)机制，让编译器来进行内存管理。

1. 使用Xcode4.2或以上版本
2. 使用LLVM编译器3.0或以上版本
3. 编译器选项中设置ARC为有效

###所有权修饰符

* __strong修饰符
* __weak修饰符
* __unsafe_unretained修饰符
* __autoreleasing修饰符

####1. __strong修饰符

__strong修饰符为id类型和对象类型默认的所有权修饰符,可以省略。

	id __strong obj = [[NSObject alloc] init];

####2. __weak修饰符

如果只用__strong修饰符，自动引用计数式内容管理必然会发生“循环引用”的问题。

__weak提供弱引用，弱引用不能持有对象实例，可以解决“循环引用”的问题。

下面这段代码：

	id __weak obj = [[NSObject alloc] init];
	NSLog(@"%@",obj);
	
会输出 nil 。因为__weak修饰的obj持用弱引用，在赋值过后，并没有对象强引用它，生成的对象会立即释放。这样obj自动赋值为nil。

再来看看下面这段代码的输出：

	id __weak obj0 = nil;
    {
        id __strong obj1 = [[NSObject alloc] init];
        obj0 = obj1;
        NSLog(@"%@",obj0);
    }
    NSLog(@"%@",obj0);
    
结果：

	<NSObject: 0x109420000>
	nil

注：__weak修饰符只能用于IOS5以上以及OS X Lion以上版本的应用程序。在它们一下的程序可使用__unsafe_unretained修饰符来代替。

####3. __unsafe_unretained修饰符

__unsafe_unretained修饰符，是不安全的所有权修饰符。有__unsafe_unretained修饰符的变量不属于编译器的内存管理对象。

	id __unsafe_unretained obj = [[NSObject alloc] init];
	NSLog(@"%@",obj);

这里和__weak一样，会输出`nil`。难道这两个修饰符相同吗？再来看第二个例子：

	id __unsafe_unretained obj0 = nil;
    {
        id __strong obj1 = [[NSObject alloc] init];
        obj0 = obj1;
        NSLog(@"%@",obj0);
    }
    NSLog(@"%@",obj0);
   
输出：
	
	<NSObject: 0x10972ced0>
	<NSObject: 0x10972ced0>

可以看到结果已经和__weak不同。

这里在第二次打印的时候，obj0已经成为一个野指针。

__weak与__unsafe_unretained的区别可以简单地说：

当弱引用的对象被释放，__weak修饰的对象会自动赋值为nil；而__unsafe_unretained修饰的对象不会。

####4. __autoreleasing修饰符

在ARC有效的时候，不能使用NSAutoreleasePool。为了实现想非ARC工程的NSAutoreleasePool该怎么办？使用__autoreleasing修饰符。

一下两段代码效果相同：

	/* NO ARC */
	NSAutoreleasePool* pool = [[NSAutoreleasePool alloc] init];
    id obj = [[NSObject alloc] init];
    [obj autorelease];
    [pool drain];

	/*  ARC  */
	@autoreleasepool {
        id __autoreleasing obj = [[NSObject alloc] init];
    }
    
但是，显式地附加__autoreleasing修饰符同显式地附加__strong一样罕见。一般不必显示地添加__autoreleasing修饰符。

<p style="color:red"> 另外，无论ARC是否有效，调试用的私有方法_objc_autoreleasePoolPrint()都可以使用。利用这个函数可以有效地帮助我们调试注册到autoreleasepool上的对象。 </p>

###ARC的规则

1\. 不能使用retain/release/retainCount/autorelease

2\. 不能使用NSAllocateObject/NSDeallocateObject

3\. 遵守内存管理的方法命名规则

4\. 不要显示调用dealloc

5\. 使用@autoreleasepool快代替NSAutoreleasePool

6\. 不能使用NSZone

7\. 对象型变量不能作为C语言结构体的成员

要把对象型变量加入到结构体成员，可强制转换为void* 或是添加`__unsafe_unretained`修饰符。

####显示转换id 和 void*

下面这段代码，在ARC下会编译错误：

	id obj = [[NSObject alloc] init];
	void* p = obj;
	
需要使用“__bridge转换”：

1、 __bridge
	
	/* ARC */
	id obj1 = [[NSObject alloc] init];
    void* p = (__bridge void*)obj1;
    
相当于：

	/* NO ARC */
	id obj = [[NSObject alloc] init];
	void* p = obj;
    
但是使用__bridge转换，其安全性与赋值__unsafe_unretained修饰符相似，甚至更低。

2、 __bridge_retained

	/* ARC */
	id obj1 = [[NSObject alloc] init];
    void* p = (__bridge_retained void*)obj1;

相当于：

	/* NO ARC */
	id obj1 = [[NSObject alloc] init];
	void* p = obj1;
	[(id)p retain];
	
下面的代码：
	
	void* p = 0;
	{
		id obj1 = [[NSObject alloc] init];
    	void* p = (__bridge_retained void*)obj1;
	}
	NSLog(@"%@",p);

由于使用__bridge_retained，p在对后持有该对象，所以会打印出该对象。

3、 __bridge_transfer

	/* ARC */
	id obj1 = [[NSObject alloc] init];
    void* p = (__bridge_transfer void*)obj1;
    
相当于：

	id obj1 = [[NSObject alloc] init];
	void* p = obj1;
	[(id)p retain];
	[obj1 release];
	
当然可是实现双向的转换。这些转换多数使用在Objec-C对象和Core Foundation对象之间的互相转换。

以下两个函数为系统提供的方法，实现Toll-Free Bridging：

	NS_INLINE id CFBridgingRelease(CFTypeRef CF_CONSUMED X) {
    return (__bridge_transfer id)X;
	}
    NS_INLINE CF_RETURNS_RETAINED CFTypeRef CFBridgingRetain(id X) {
    return (__bridge_retained CFTypeRef)X;
	}

[Toll-Free Bridging](https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaEncyclopedia/Toll-FreeBridgin/Toll-FreeBridgin.html)

###属性

ARC中，属性声明的属性与所有权修饰符的关系


   属性声明   | 所有权修饰符           
:---------------:| :------------------: 
assign           | __unsafe_unretained  
copy             | __strong(复制对象)    
retain           | __strong             
strong           | __strong             
unsafe_unretained| __unsafe_unretained 
weak             | __weak               

<p style="color:red">有一种情况不能使用__weak修饰符：</p> 

 	- (BOOL)allowsWeakReference UNAVAILABLE_ATTRIBUTE;
	- (BOOL)retainWeakReference UNAVAILABLE_ATTRIBUTE;
	
如果NSobject实例上面的两个方法返回NO，绝对不能使用__weak修饰符。