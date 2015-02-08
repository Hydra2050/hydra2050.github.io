---
layout: post
title: "如何创建和使用私有cocoapods repo"
date: 2015-02-08 15:22:36 +0800
comments: true
categories: 
---

## 安装cocoapods

	gem update --system
	gem sources -l
	gem install cocoapods
	pod setup

## 创建私有cocoapods repo

### 第一步，创建私有Spec Repo

在服务器创建repo裸仓库：

	mkdir REPO_NAME.git
	cd REPO_NAME.git
	git init --bare

### 第二步，将私有pod repo添加到本地

	pod repo list
	pod repo add REPO_NAME SOURCE_URL
	
### 第三步，检测pod repo是否正确

	pod repo lint REPO_NAME
	
## 为私有pod添加新的Spec

### 第一步，创建podspec文件

	pod spec create SPEC_NAME
	
### 第二步，修改Spec文件


### 第三步，检测Spec文件是否正确

	pod spec lint SPEC_NAME
	
### 第四步，上传Spec文件至私有pod repo

	pod repo push REPO_NAME SPEC_NAME.podspec
	
因为我使用的时`ssh`URL获取，会产生`warning`：

	    - WARN  | [source] Git SSH URLs will NOT work for people behind firewalls configured to only allow HTTP, therefore HTTPS is preferred.
	
可以忽略这个警告：

	pod repo push --allow-warnings REPO_NAME SPEC_NAME.podspec
	
## spec lint 技巧

在执行`pod spec line xxx`的时候出错，但提示往往比较简单，可以添加参数`--no-clean`：

	pod spec lint --no-clean xxx
	
执行完成后，根据提示的路径，打开lint生成的工程文件，编译查看出错的原因。

一般路径为：`/private/tmp/CocoaPods/Lint/Pods/Pods.xcodeproj`。


参考链接：[http://guides.cocoapods.org/making/private-cocoapods.html](http://guides.cocoapods.org/making/private-cocoapods.html)
