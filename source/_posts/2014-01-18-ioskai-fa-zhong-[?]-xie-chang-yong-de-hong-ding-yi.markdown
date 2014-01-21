---
layout: post
title: "iOS开发中一些常用的宏定义"
date: 2014-01-18 21:55:51 +0800
comments: true
categories: 
---

Debug输出：

	#ifdef DEBUG
  		#define debug(format, ...) CFShow([NSString stringWithFormat:format, ## __VA_ARGS__]);
  		#define debugAlert(format, ...)  {UIAlertView *alert = [[UIAlertView alloc] initWithTitle:[NSString stringWithFormat:@"%s\n line: %d ", __PRETTY_FUNCTION__, __LINE__] message:[NSString stringWithFormat:format, ##__VA_ARGS__]  delegate:nil cancelButtonTitle:@"Done" otherButtonTitles:nil]; [alert show]; [alert release];}
	#else
  		#define debug(format, ...) 
  		#define debugAlert(format, ...)
	#endif
	
	
检测是否支持ARC：

	#if __has_feature(objc_arc)
  		// ARC
	#else
  		// No ARC
	#endif
	
http://iosdevelopertips.com/debugging/remove-debug-code-for-release-build.html

http://iosdevelopertips.com/debugging/display-debug-information-in-uialertview.html

