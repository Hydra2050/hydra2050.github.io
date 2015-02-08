---
layout: post
title: "如何提高pod install执行速度"
date: 2015-02-08 14:05:04 +0800
comments: true
categories: 
---

应用集成了cocoapods一段时间，发现每次执行`pod insgtall`命令都会比较慢，有时要等十多分钟甚至更久，严重影响工作效率和开发的心情。

## `pod install`执行慢的原因

Cocopods的`pod install`命令一般分两步：

1. 执行`pod repo update`，更新本地的repo Spec文件（[https://github.com/CocoaPods/Specs.git](https://github.com/CocoaPods/Specs.git)）
2. 根据spec的配置，获取源文件，一般是通过git clone的方式从github上下载第三方库

由于`GWF`的存在，经常存在github无法访问的情况；有时可以访问，速度也比较慢。人生苦短！！

可以针对这两方面进行提高`pod install`执行速度。

## 减少pod repo update的次数

执行`pod repo update`是必要的，保证每次安装的时候，能够获取最新的Spec文件。但不必每次都执行，建议每天执行一次。在执行`pod install`的时候添加参数，不再执行`pod repo update`：

	pod install --no-repo-update


## 使用本地的git mirror解决下载慢的问题


解决了repo更新的问题，仍然存在一系列问题：

* 将源文件克隆到本地速度缓慢
* 一旦被托管在github上的第三方库被删除掉，我们将再也无法获取到第三方库
* 如果访问外网的网络暂时性的中断，无法执行`pod install`，影响正常开发

为了解决以上问题，在公共服务器上搭建了一个GitMirror，为每一个使用到的第三方库创建一个mirror仓库。以后每次执行install的时候，实际都是从本地服务器下载，速度就会有明显的提升。

### 修改本地spec配置

第一步要将第三库的源文件下载地址修改为本地服务器。写了一个脚本`moveToGitMirror.py`自动完成本地spec配置修改。执行命令如下：

	cd ~/.cocoapods/repos/master/
	python moveToGitMirror.py
	
这个脚本的功能很简单，遍历当前目录下的文件，查找后缀名为.spec.json的文件，查找到source路径，然后修改为镜像地址。

这里需要注意的是，一旦修改，将完全依赖镜像的服务器。可以替换成一个指定的域名，而不是一个固定的ip地址。这样通过修改本地`hosts`文件，实现source路径的整体切换。

### 为一个第三方库创建git mirror

如果项目中需要一个第三方库，以`Masonry`为例。

	ssh root@xx.xx.xx.xx
	cd GitRepoMirror/
	git clone --mirror https://github.com/Masonry/Masonry.git

### 设置免密码远程访问

完成以上的操作，执行`pod install`，会发现下载一个第三方库都需要我们输入一次密码。what？？？太烦了！

为了实现免密码远程访问，我们需要把本机的rsa公钥放入远程服务器的`authorized_keys`文件下。如果本机没有生产过rsa密钥，执行以下命令：

	ssh-keygen -t rsa -C "xxx.com"

## 下一步

下一步，可以创建一个属于自己的私有pod repo了，以后再介绍。
