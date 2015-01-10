---
layout: post
title: "UIImagePickerController cause idleTimerDisabled setting problem"
date: 2015-01-10 10:48:27 +0800
comments: true
categories: 
---

前一段时间，遇到了一个奇怪的问题：在程序里设置了`idleTimerDisabled`为`YES`，但是还是有用户反馈，程序会自动锁屏。测试也曾反馈，曾经遇到过这个问题，但一直未找到重现方法。在Stackoverflow看到，与UIImagePickerController的使用有关。

## 重现

设置`idleTimerDisabled`为`YES`，创建UIImagePickerController，设置sourceType为`UIImagePickerControllerSourceTypeCamera`，然后弹出拍照页面。拍照页面关闭后，等待自动锁屏的出现。


## 分析

设置断点:

	-[UIApplication setIdleTimerDisabled:]
	
![idleTimer breakpoint](/images/20150110_1.png)
	
运行后会发现，在UIImagePickerController弹出的时候，进入一次断点。使用LLDB命令(因为我用的是arm64的机器，所以寄存器为x，如果是armv7，为r)：

	po $x0
	<UIApplication: 0x156e008e0>
	
	p (SEL)$x1
	(SEL) $1 = "setIdleTimerDisabled:"

	po $x2
	1
	
在弹出UIImagePickerController的时候，系统会自动将`idleTimerDisabled`设置为`YES`。

继续运行，关闭UIImagePickerController，由于这只的animation为`YES`，会进入两次断点。两次的参数相同，都为`nil`。

	po $x2
	nil
	
这就会将`idleTimerDisabled`设置为`NO`，导致之前设置的值被修改。

	
## 解决

目前想到的解决方法，在UIImagePickerController消失后，重新设置idleTimerDisabled为需要的值。

	#pragma mark - UIImagePickerControllerDelegate

	- (void)imagePickerController:(UIImagePickerController *)picker 		didFinishPickingMediaWithInfo:(NSDictionary *)info
	{
    	[self dismissViewControllerAnimated:YES completion:^{
        [self performSelector:@selector(resetIdleTimerDisabled) withObject:nil 			afterDelay:1.0];
   		 }];
	}

	- (void)imagePickerControllerDidCancel:(UIImagePickerController *)picker
	{
  	  [self dismissViewControllerAnimated:YES completion:^{
 	  [self performSelector:@selector(resetIdleTimerDisabled) withObject:nil 		afterDelay:1.0];
 	   }];
	}
	
参考：

[http://stackoverflow.com/questions/23391564/ios-idletimerdisabled-yes-works-only-until-imagepicker-was-used](http://stackoverflow.com/questions/23391564/ios-idletimerdisabled-yes-works-only-until-imagepicker-was-used)

[http://www.objc.io/issue-19/](http://www.objc.io/issue-19/)
