---
layout: post
title: "Object-c中的黑魔法——Method Swizzling"
date: 2013-12-08 09:46:50 +0800
comments: true
categories: 
---

上一篇讲了[Object-c的函数调用机制](http://hydra2050.github.io/blog/2013/12/04/object-cde-han-shu-diao-yong-ji-zhi/)，对理解Method Swizzling有很大帮助。这一次来看一看Object-c中的黑魔法——Method Swizzling。

##Method Swizzling介绍

什么是Method Swizzling？简单来讲，就是修改SEL与IMP之间的映射关系，达到修改最终执行方法的目的。Method Swizzling，听起来就有黑魔法的意味。

##如何实现Method Swizzling

###1.创建一个category
	
	#import <Foundation/Foundation.h>  
	@interface UIView (MyAdditions)   
	@end  
	
###2.创建一个swizzling函数

	@implementation UIView (MyAdditions)
	
	- (void)my_setFrame:(CGRect)frame {
	    // custom function
	    [self my_setFrame:frame];
	}
	@end

等等，这不是循环调用吗？别忘了，我们后面会施展黑魔法，改变IMP,调用的不再是自己，不会有循环调用的问题。

###3.交换IMP

<p style="color:red">注意别忘记引入：#import <objc/runtime.h>

	Method oriMethod =  class_getInstanceMethod([UIView class], @selector(setFrame:));  
    Method myMethod = class_getInstanceMethod([UIView class], @selector(my_setFrame:));  
    method_exchangeImplementations(oriMethod, myMethod); 

这样调用 [UIView setFrame:] 时，实际调用的是自定义的方法 [UIView my_setFrame:]。


##Method Swizzling的利与弊

这里有非常好的答案：

[http://stackoverflow.com/questions/5339276/what-are-the-dangers-of-method-swizzling-in-objective-c](http://stackoverflow.com/questions/5339276/what-are-the-dangers-of-method-swizzling-in-objective-c)

其中给出了一个解决命名冲突的定义方式：

	@implementation NSView (MyViewAdditions)
	static void MySetFrame(id self, SEL _cmd, NSRect frame);
	static void (*SetFrameIMP)(id self, SEL _cmd, NSRect frame);
	
	static void MySetFrame(id self, SEL _cmd, NSRect frame) {
	   // do custom work
	   SetFrameIMP(self, _cmd, frame);
	}
	
	+ (void)load {
	   [self swizzle:@selector(setFrame:) with:(IMP)MySetFrame store:(IMP *)&SetFrameIMP];
	}
	@end
	
	typedef IMP *IMPPointer;
	BOOL class_swizzleMethodAndStore(Class class, SEL original, IMP replacement, IMPPointer store) {
	    IMP imp = NULL;
	    Method method = class_getInstanceMethod(class, original);
	    if (method) {
	        const char *type = method_getTypeEncoding(method);
	        imp = class_replaceMethod(class, original, replacement, type);
	        if (!imp) {
	            imp = method_getImplementation(method);
	        }
	    }
	    if (imp && store) { *store = imp; }
	    return (imp != NULL);
	}
	
	@implementation NSObject (FRRuntimeAdditions)
	+ (BOOL)swizzle:(SEL)original with:(IMP)replacement store:(IMPPointer)store {
	    return class_swizzleMethodAndStore(self, original, replacement, store);
	}
	@end

##总结

对于我个人而言，并不是非常喜欢这种黑魔法。感觉还是慎用为妙，如果使用，尽量仅在load中swizzling。另外，网上有人说使用了Method Swizzling无法通过appstore的审核，还没有验证过。
虽然不建议发布使用，但是其他地方还是有很多用武之地的，比如调试、代码分析等，可以自由想像。

这里有一个[MethodSwizzlingDemo](https://github.com/Hydra2050/MethodSwizzlingDemo)，通过Method Swizzling，描绘每一个View的border。效果图：



