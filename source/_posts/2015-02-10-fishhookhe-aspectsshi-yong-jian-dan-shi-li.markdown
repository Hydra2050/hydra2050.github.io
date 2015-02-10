---
layout: post
title: "fishhook和Aspects使用简单示例"
date: 2015-02-10 10:27:11 +0800
comments: true
categories: 
---

[fishhook](https://github.com/facebook/fishhook): A library that enables dynamically rebinding symbols in Mach-O binaries running on iOS.

	#import <dlfcn.h>
	#import "fishhook.h"

	static OSStatus (*origin_AudioSessionSetProperty)(unsigned long, unsigned long, const void*);

	void save_original_symbols() {
    	origin_AudioSessionSetProperty = dlsym(RTLD_DEFAULT, "AudioSessionSetProperty");
	}

	OSStatus my_AudioSessionSetProperty(unsigned long inID, unsigned long inDataSize, const void *inData){
   	    printf("Calling real AudioSessionSetProperty: inID %ld  \n", inID);
    
    	return origin_AudioSessionSetProperty(inID, inDataSize, inData);
	}

	int main(int argc, char * argv[]) {
   	 @autoreleasepool {
     	   save_original_symbols();
       	   rebind_symbols((struct rebinding[]){{"AudioSessionSetProperty",my_AudioSessionSetProperty}}, 1);
        
      	   return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
   		 }
	}
	
# Aspects

[Aspects](https://github.com/steipete/Aspects.git): Delightful, simple library for aspect oriented programming.



	[[AVAudioSession sharedInstance] aspect_hookSelector:@selector(setCategory:withOptions:error:) withOptions:0 usingBlock:^(id<AspectInfo> info,NSString *category, AVAudioSessionCategoryOptions option, NSError ** error){
        NSLog(@"AVAudioSession set category with category : %@", category);
    }error:nil];
    
需要注意的一点，参数`block`需要与hook的函数相符，如果不匹配，会有如下提示：

	Aspects: Block signature <NSMethodSignature: 0x7f93b8c35c80> doesn't match <NSMethodSignature: 0x7f93b8c33c30>.
    