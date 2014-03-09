---
layout: post
title: "使用UIMotionEffect实现设备的倾斜视差"
date: 2014-02-16 18:28:08 +0800
comments: true
categories: 
---

iOS7更新后，当倾斜手机的时候，会看到屏幕上的背景图片会随着手机的倾斜度而移动，产生视差。刚开始更新的时候，还以为是自己的错觉，－－！。

UIMotionEffect就像它的名字一样，处理motion effect。UIMotionEffect是一个抽象的基类，子类通过重写`keyPathsAndRelativeValuesForViewerOffset:`方法，当检测到动作的时候

苹果提供了一个子类UIInterpolatingMotionEffect，通过它我们可以实现对设备水平和竖直方向倾斜。

	UIInterpolatingMotionEffect *horizontalMotionEffect = [[UIInterpolatingMotionEffect alloc] initWithKeyPath:@"center.x" type:UIInterpolatingMotionEffectTypeTiltAlongHorizontalAxis];
	horizontalMotionEffect.minimumRelativeValue = @(-50);
	horizontalMotionEffect.maximumRelativeValue = @(50);
	[redView addMotionEffect:horizontalMotionEffect];
	


http://www.teehanlax.com/blog/introduction-to-uimotioneffect/
https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIMotionEffect_class/Reference/Reference.html#//apple_ref/doc/uid/TP40013376
https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIMotionEffectGroup_class/Reference/Reference.html#//apple_ref/doc/uid/TP40013378
