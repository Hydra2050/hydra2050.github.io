---
layout: post
title: "CocoaPods：Objective-C的依赖管理工具"
date: 2013-12-11 21:31:32 +0800
comments: true
categories: 
---

##引子

在IOS开发中，工程中会用到很多第三方的开源库，像ASIHttpRequest、AFNetworking、JSONKit等等。要添加一个第三方库的：

1. 先查找并下载到本地，然后添加到工程
2. 添加第三方开源库所依赖的framework
3. 一些第三方开源库需要修改OtherLinerFlag的参数

一旦开始一个新的项目，这些工作又要重新做一遍。另外还需要自己管理第三方库更新的问题。

有没有一个能够帮助我们自动完成这一系列工作的方法呢？没错，就是今天的主角——CocoaPods。

##CocoaPods介绍

CocoaPods为你的Xcode工程管理库的依赖。你的工程的第三方库依赖由一个单独的叫做Podfile的文本文件管理。CocoaPods可以获取第三方库源代码、解决库的依赖，然后将你的工程与它放在一个新创建的workspace中。最终的目标是通过创建一个更集中的生态系统，以提高第三方的开源库可发现性和参与性。

官网地址：[http://cocoapods.org/](http://cocoapods.org/)

GitHub地址：[https://github.com/cocoapods/cocoapods](https://github.com/cocoapods/cocoapods)

##安装CocoaPods

CocoaPods使用Ruby创建，需要预先安装ruby。Mac下自默认安装好了ruby，可以通过命令查看：
	
	ruby --version
	
使用ruby的gem命令安装：

	sudo gem install cocoapods
	pod setup
	
##更新CocoaPods

更新已安装的CocoaPods，只需要输入：

	sudo gem update cocoapods
	
参见：[CocoaPods Getting Started](http://guides.cocoapods.org/using/getting-started.html)

##使用CocoaPods查找第三方库

CocoaPods可以很方便地查找第三方库：

	pod search ReactiveCocoa
	


##将CocoaPods添加到Xcode工程

创建一个[Podfile](http://guides.cocoapods.org/using/the-podfile.html)文件，添加你所需要的第三方库文件：
	
	platform :ios, '6.0'
	pod 'AFNetworking', '~> 2.0'  
	pod 'ObjectiveSugar', '~> 0.5'
	
保存并放在工程的根目录，然后：
	
	pod install
	
这样就会生成一个MyApp.xcworkspace，以后经通过使用这个打开工程，而不是原来的MyApp.xcodeproj.

如果需要更新工程使用的第三方库，只需要编辑Podfile文件，然后再次执行pod install 即可。

##集成到一个已存在的workspace中

如果想将CocoaPods集成到一个已经存在的workspace中，只需要在Podfile中增加一句：

	workspace 'MyWorkspace'
	
##这一系列命令后到底做了什么

使用pod install命令，实际做了：

1. 创建或者更新workspace
2. 将你的工程添加到这个workspace中
3. 如果需要将CocoaPods静态库工程添加到workspace中
4. 添加libPods.a
5. 将CocoaPods的Xocde configurations file添加到你的工程
6. 将你的工程的target configurations改变为急于CocoaPods的
7. 添加一个编译参数来拷贝pods中的资源文件到你的app bundle。i.e. a ‘Script build phase’ after all other build phases with the following:
Shell: /bin/sh
Script: ${SRCROOT}/Pods/PodsResources.sh

参见：[Using CocoaPods](http://guides.cocoapods.org/using/using-cocoapods.html#what-is-happening-behind-the-scenes?)

##总结

使用CocoaPods管理第三方库，能够帮助完成很多之前繁琐的工作，提高工作效率。第三方库的更新更是省心了不少，不再需要再一个一个查看，哪个第三方库更新了没有。

另外有一款APP，[Podlife](http://davander.com/podlife.html)，让我们了解Pods上第三方库更新的情况，很不错。

