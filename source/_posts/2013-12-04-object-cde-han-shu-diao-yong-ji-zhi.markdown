---
layout: post
title: "Object-c的函数调用机制"
date: 2013-12-04 22:01:26 +0800
comments: true
categories: 
---

##起因

有一次，被问到，下面这段代码执行，会得到什么结果：

	@interface ClassA : NSObject  
	-(void) foo;  
	@end  
	@implementation ClassA  
	-(void) foo  
	{  
		NSLog(@"fork Class A's function");  
	}  
	@end  
  
	@interface ClassB : NSObject  
	-(void) foo;  
	@end  
	@implementation ClassB  
	-(void) foo  
	{  
		NSLog(@"fork Class B's function");  
	}  
	@end  
	
	ClassA* a = [[ClassA alloc] init];  
	[a foo];  
	ClassB* b = (ClassB*)a;  
	[b foo];
	
说来惭愧，这么久竟然不清楚object-c的函数调用机制，于是网上查阅了一些资料，整理出一篇文章。

##Object-C的函数调用的机制  

OC的方法调用与C++不同： Ｃ++中的方法调用可能是动态的，也可能是静态的；而ObjC中的函数调用都为动态的，通过消息传递的方式调用。

首先通过command＋shift＋o，查找NSObject类，可以看到类的定义：

	@interface NSObject <NSObject> {  
    Class isa  OBJC_ISA_AVAILABILITY;  
	}  
	
	typedef struct objc_selector *SEL;  
	
	typedef id (*IMP)(id, SEL, ...);  

在object-c中，每个对象都有一个isa成员，指向自己的类结构体，这也是object-c自省的特性实现。
为了理解消息调用机制，需要理解以上三个定义：Class   SEL   IMP

### Class  SEL IMP

先来看一看Class的定义：

	struct objc_class {  
    Class isa  OBJC_ISA_AVAILABILITY;  
  
	#if !__OBJC2__  
    Class super_class                                        OBJC2_UNAVAILABLE;  
    const charchar *name                                         OBJC2_UNAVAILABLE;  
    long version                                             OBJC2_UNAVAILABLE;  
    long info                                                OBJC2_UNAVAILABLE;  
    long instance_size                                       OBJC2_UNAVAILABLE;  
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;  
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;  
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;  
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;  
	#endif  
  
	} OBJC2_UNAVAILABLE; 
	
其中我们比较关心的是参数列表ivars，方法列表methodLists，协议列表protocols。当一个类有调用方法时：

	[receiver message]

会调用消息函数，objc_msgSend：

	objc_msgSend(receiver, selector)

如果有参数要传递的话：

	objc_msgSend(receiver, selector, arg1, arg2, ...)
	
![Selector](./images/messaging.jpg)
	
当一个消息传递给一个对象的时候，objc_msgSend根据对象的isa指针，到类结构体methodLists链表找指定selector，如果没有，向父类寻找，知道找到NSObject类位置。这样就实现了动态方法调用。

那么IMP又是什么呢？
简单来说，就是一个函数指针，指向SEL的方法的地址。NSObject 类中的methodForSelector：方法就是这样一个获取指向方法实现IMP 的指针。
下面的例子展示了怎么使用指针来调用setFilled:的方法实现：

	void (*setter)(id, SEL, BOOL);  
	int i;  
	setter = (void (*)(id, SEL, BOOL))[target methodForSelector:@selector(setFilled:)];  
	for ( i = 0 ; i < 1000 ; i++ )  
    	setter(targetList[i], @selector(setFilled:), YES); 
    	
查找IMP 时：

1. 首先去该类的方法 cache 中查找，如果找到了就返回它；

2. 如果没有找到，就去该类的方法列表中查找。如果在该类的方法列表中找到了，则将 IMP 返回，并将它加入cache中缓存起来。根据最近使用原则，这个方法再次调用的可能性很大，缓存起来可以节省下次调用再次查找的开销。

3. 如果在该类的方法列表中没找到对应的IMP，在通过该类结构中的super_class指针在其父类结构的方法列表中去查找，直到在某个父类的方法列表中找到对应的IMP，返回它，并加入cache中。

4. 如果在自身以及所有父类的方法列表中都没有找到对应的 IMP，则进入下文中要讲的消息转发流程。
	
###消息转发

通常，给一个对象发送它不能处理的消息会得到出错提示，然而，Objective-C运行时系统在抛出错误之前，会给消息接收对象发送一条特别的消息forwardInvocation来通知该对象，该消息的唯一参数是个NSInvocation类型的对象——该对象封装了原始的消息和消息的参数。

我们可以实现forwardInvocation:方法来对不能处理的消息做一些默认的处理，也可以将消息转发给其他对象来处理，而不抛出错误。

NSObject 的forwardInvocation:方法默认的调用doesNotRecognizeSelector:方法

	0   CoreFoundation                      0x01a1c5e4 __exceptionPreprocess + 180  
	1   libobjc.A.dylib                     0x015728b6 objc_exception_throw + 44  
	2   CoreFoundation                      0x01ab9903 -[NSObject(NSObject) doesNotRecognizeSelector:] + 275  

重写forwardInvocation：

	-(void) helloWorld
	{  
   		 NSLog(@"Hello world!");
   	}  
	- (void) forwardInvocation:(NSInvocation *)anInvocation  
	{  
     	if([self respondsToSelector:@selector(helloWorld)])  
    	 {  
         	  [self helloWorld];  
    	 }  
     	else  
   		{  
         	[super forwardInvocation:anInvocation];  
     	}  
	}  
	- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector  
	{  
   		if ([self respondsToSelector:aSelector]) {  
        	return [super methodSignatureForSelector:aSelector];  
    	}  
    	else {  
        	return [NSMethodSignature signatureWithObjCTypes:"v@:"];  
    	}  
	}  
	
执行结果：

	2013-11-23 15:23:50.464 VCTransitionDemo[1272:70b] Hello world! 
	

<p style="color:red">注意：我们重写forwardInvocation方法的时候，还必须同时重写methodSignatureForSelector方法，该方法返回表示一个方法的字符串，具体如何使用请看Type Encodings。</p>


##小结

到这里，应该能够回答最初的那个问题了。

OC扩展了标准的ANSI C编程语言，将Smalltalk式的消息传递机制加入到ANSI C中。它的方法调用与C/C++还是有很大差别的。

接下来，会再深入OC的方法调用，写一些关于Method Swizzling的内容。

参考资料：

[Objective-C Runtime Reference](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtHowMessagingWorks.html#//apple_ref/doc/uid/TP40008048-CH104-SW1)

[Type Encodings](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1)
