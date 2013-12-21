---
layout: post
title: "IOS7 New Feature: iBeacon"
date: 2013-12-21 18:32:32 +0800
comments: true
categories: 
---

##前言

iBeacon作为IOS7的一项新特性，在今年的WWDC大会上正式正式发布。虽然官方对它的描述并不多，WWDC的视频中也没有一个是讲解iBeacon的，但是在WWDC2013的Keynote上还是看到了它的身影。

![WWDC2013 Keynote](/images/201312211904.jpeg)

iBeacon也是我今年WWDC上最喜欢的新特性之一，因为我一直认为更便捷、去中心化的沟通才是未来。在IOS7推出Beta版的时候，我也曾经实践过新推出的Nearby Networking With Multipeer Connectivity。但是相比之下，iBeacon要优秀很多。


目前有不少硬件厂商在生产基于iBeacon的基站，Estimote很早就推出了Estimote Beacons，利用苹果的iBeacon技术，实现Distance、Proximity和Notification三种功能。如果想体验一下，可以在Appstore搜索下载应用Estimote。同时Estimote Beacons提供了一套API，可以方便地进行编程。

[Estimote Beacons的API地址](https://github.com/Estimote/iOS-SDK)

![Estimote Beacons](/images/201312211906.jpeg)

##What is iBeacon?

iBeacon通过使用低功耗蓝牙技术（Bluetooth Low Energy，也就是通常所说的Bluetooth 4.0或者Bluetooth Smart），可以创建一个信号区域，当设备进入该区域时，相应的应用程序便会提示用户是否需要接入这个信号网络。通过能够放置在任何物体中的小型无线传感器和低功耗蓝牙技术，用户便能使用iPhone来传输数据。

![iBeacon](/images/201312211907.jpeg)

##什么是低功耗蓝牙技术？

低功耗蓝牙技术的最大特点便在于低功耗，从而能使设备拥有更长的续航时间。不过低功耗蓝牙技术仅支持较低的文件传输速率，因此可以用于可穿戴式智能设备之间的信息传送，但却不能完成像传输音频这样的任务。

##一个简单的Demo

[Demo下载地址](https://github.com/Hydra2050/BeaconDemo)

###Start a BeaconBroadcast

1、 创建一个BeaconRegion，作为Beacon基站的广播的Region对象。其中UUID为Beacon基站的唯一标识。

	NSUUID* myUUID = [[NSUUID alloc] initWithUUIDString:@"E621E1F8-C36C-495A-93FC-0C247A3E6E5D"];
    self.targetBeaconRegion = [[CLBeaconRegion alloc] initWithProximityUUID:myUUID identifier:@"My Beacon Demo 20131221"];
    
2、 创建一个CBPeripheralManager对象

	 self.peripheraManager = [[CBPeripheralManager alloc] initWithDelegate:self queue:nil options:nil];
	 
3、 开始广播

	NSDictionary* peripheraData = [self.boardcastBeaconRegion peripheralDataWithMeasuredPower:nil];
	[self.peripheraManager startAdvertising:peripheraData];

开始广播后，可以处理一下回调函数，如：
	
	- (void)peripheralManagerDidUpdateState:(CBPeripheralManager *)peripheral
	- (void)peripheralManagerDidStartAdvertising:(CBPeripheralManager *)peripheral error:(NSError *)error
	
###Start a BeaconMonitor

1、 创建一个BeaconRegion，作为搜索的Region对象。其中UUID为Beacon基站的唯一标识。
	
	NSUUID* myUUID = [[NSUUID alloc] initWithUUIDString:@"E621E1F8-C36C-495A-93FC-0C247A3E6E5D"];
    self.targetBeaconRegion = [[CLBeaconRegion alloc] initWithProximityUUID:myUUID identifier:@"My Beacon Demo 20131221"];

2、 创建一个CLLocationManager对象

	self.locationManager = [[CLLocationManager alloc] init];
    self.locationManager.delegate = self;

3、 开始搜索Beacon基站

	[self.locationManager startRangingBeaconsInRegion:self.targetBeaconRegion];

接下来可以处理一下回调函数，如：

	- (void)locationManager:(CLLocationManager *)manager
        didRangeBeacons:(NSArray *)beacons inRegion:(CLBeaconRegion *)region
	- (void)locationManager:(CLLocationManager *)manager
	 didUpdateLocations:(NSArray *)locations
	- (void)locationManager:(CLLocationManager *)manager
         didEnterRegion:(CLRegion *)region
    - (void)locationManager:(CLLocationManager *)manager
          didExitRegion:(CLRegion *)region
	- (void)locationManager:(CLLocationManager *)manager didChangeAuthorizationStatus:(CLAuthorizationStatus)status
	
效果图：

![CLProximityImmediate](/images/201312211958.PNG)

![CLProximityNear](/images/201312211959.PNG)

![CLProximityFar](/images/201312212001.PNG)

![CLProximityUnknown](/images/201312212002.PNG)

![didExitRegion](/images/201312212003.PNG)

##总结

通过Demo，可以对iBeacon有一个简单的了解。iBeacon可以方便实现室内导航、商品消息推送、支付等功能。网上很多文章认为iBeacon会成为NFC杀手。到底iBeacon能发展到什么地步，让我们拭目以待吧。

