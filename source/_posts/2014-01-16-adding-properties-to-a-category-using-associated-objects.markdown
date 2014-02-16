---
layout: post
title: "Adding Properties to a Category Using Associated Objects"
date: 2014-01-16 22:39:50 +0800
comments: true
categories: 
---

##Object-C中的Category

Object-C中的Category，相信大家一定不会陌生。一般来说，我们可以用它来为一个类添加新的方法：

	@interface NSString (NumberUtils)
	- (BOOL)isNumeric; 
	@end

	@implementation NSString (NumberUtils) 
	- (BOOL)isNumeric
	{
    	NSScanner *scanner = [NSScanner scannerWithString:self];
    	return [scanner scanFloat:NULL]? [scanner isAtEnd]: NO;
	}
	@end
	
我们也可以Category中定义新的数据成员，但是不能够对它们使用synthesize。这是因为一个类的结构已经在编译器决定了，而Category是在运行期定义生成的，这样就没有办法改变类结构体中的ivars。

	struct objc_class {  
		Class isa  OBJC_ISA_AVAILABILITY;  

		#if !__OBJC2__  
		Class super_class                                        OBJC2_UNAVAILABLE;  
		const charchar *name                                     OBJC2_UNAVAILABLE;  
		long version                                             OBJC2_UNAVAILABLE;  
		long info                                                OBJC2_UNAVAILABLE;  
		long instance_size                                       OBJC2_UNAVAILABLE;  
		struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;  
		struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;  
		struct objc_cache *cache                                 OBJC2_UNAVAILABLE;  
		struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;  
	#endif  

	} OBJC2_UNAVAILABLE; 	
	
可以向下面这样定义：

	@interface NSString (NumberUtils)
	@property (nonatomic, readonly, getter=isNumeric) BOOL numeric;
	@end
	
如果我们想为一个类真正添加一个数据成员，要怎么办呢？当然你可能会想到使用修饰者，封装出一个对象，包含这个类和想要添加的数据成员。但是那样要麻烦一些。

##使用Associated Objects为一个分类添加数据成员

使用Runtime来解决这个问题。要使用的API：

	 // 使用一个key值和关联策略来为一个对象设置一个关联的值
	void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy);

`objc_setAssociatedObject`：使用一个key值和关联策略来为一个对象设置一个关联的值。类似NSDictionry，一个key值，对应一个value值。

key值必须是进程的生命周期内一个唯一、不变的id值。可以使用一个NSString对象作为一个key值。但是如果试图通过一个相同的字符串值，但是不同内存地址作为key值，可能不会获得预期的结果。一个更好的选择是定义一个static的指针作为key值。

下面一个例子：

	@interface UIImage (Tagged)
	@property (nonatomic, copy) NSString *tag;
	@end	
	
	#import <objc/runtime.h> 
 
	static const void *ImageTagKey = &ImageTagKey;
	@implementation UIImage (Tagged)
 
	- (void)setTag:(NSSting *)tag
	{
    	objc_setAssociatedObject(self, ImageTagKey, tag, OBJC_ASSOCIATION_COPY_NONATOMIC);
	}
 
	- (NSString *)tag
	{
   		return objc_getAssociatedObject(self, ImageTagKey);
	}
	@end

这个例子中有一点点需要主要：

* key值使用一个指针类型static const void * 。我们必须让这个指针初始化，否则它的值就会为NULL，但是我们不关心具体指向什么，只要它是唯一的，不变的。这里面，指针指向了自己，一个唯一、不变的值。

参考文章：

[Objective-C Runtime Reference](https://developer.apple.com/library/ios/documentation/cocoa/reference/ObjCRuntimeRef/Reference/reference.html#//apple_ref/doc/constant_group/Associative_Object_Behaviors)

[Adding Properties to a Category Using Associated Objects](http://iosdevelopertips.com/objective-c/adding-properties-category-using-associated-objects.html)

[Associated Objects](http://nshipster.com/associated-objects/)


