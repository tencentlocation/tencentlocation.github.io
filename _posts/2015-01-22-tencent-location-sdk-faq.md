---
layout: post
category: "faq"
title:  "腾讯定位SDK FAQ"
tags: [定位SDK]
summary: "腾讯定位SDK常见使用问题"
---
本文总结了腾讯定位SDK常见的使用问题

# 1. 遇到问题
## 1.1 使用问题
1. 首先确认Demo有没同样问题？没有的话，Demo是怎么写的，跟您的代码有什么不同？
2. 看看[参考文档](http://open.map.qq.com/geosdk/doc/) (文档表述得不清楚的位置也欢迎向我 qq 410063005 提出)
3. QQ群提问或 在[官方论坛](http://bbs.map.qq.com/forum-47-1.html) Android SDK 版块发贴求助

## 1.2 疑似bug
如果遇到定位SDK引起app崩溃，或产生一些跟预期完全不符的行为怀疑是bug，请向我们(QQ 86147470、410063005)发送 <font color="red">问题现象描述 + 错误日志</font>，并且告知<font color="red">您的调用方式或发送代码片断</font>

# 2. FAQ
## 我要混淆代码，怎么配置?
注意不要混淆 native 方法

	-keepclasseswithmembernames class * {
	    native <methods>;
	}

如果使用的 Android SDK版本低于18，混淆时可能出现  `android.location.Location` 相关的错误，添加以下proguard配置：

	-dontwarn android.location.Location
	
定位SDK从4.0版本开始还需要添加以下配置

	-dontwarn  org.eclipse.jdt.annotation.**
	
完整的配置如下：

```
-keepclasseswithmembernames class * {
  native <methods>;
}
-dontwarn android.location.Location
-dontwarn  org.eclipse.jdt.annotation.**
```
	
## GPS定不了位？
关于GPS应有以下几个基本认识：

1. GPS只能得到坐标值，需要通过网络才能根据坐标值得到地址信息
2. GPS首次定位非常可能非常耗时，不同机器上差异较大
3. GPS在信号好的情况下才能定位，一般要求在室外、无遮挡；一些机器在窗户边有可能可以成功定位。

## 定位误差有时上千米，正常吗？
定位存在精度问题，精度由高到低如下：

	GPS > Wi-Fi > 基站
	
在不开 GPS 和 Wi-Fi 的情况下，如果遇到覆盖范围相当大的基站，定位精度将非常低。

注：<font color="red">开 Wi-Fi 指打开 Wi-Fi 开关即可，并不要求一定连接到某个热点</font>。如果您的 app 对精度要求较高，建议 app 提示用户打开 Wi-Fi 以提高定位精度(类似于地图类 app 提示用户打开 GPS)。

## 为什么模拟器上无法定位?
模拟器不能得到基站和Wi-Fi热点信息，所以不能定位。<font color="red">请在真实设备上使用定位SDK进行开发</font>。

## 怎样区分是"GPS定位"还是"网络定位"?

	TencentLocation location = ...;
	String provider = location.getProvider();
	if (TencentLocation.GPS_PROVIDER.equals(provider)) {
		// location 是GPS定位结果
	} else if (TencentLocation.NETWORK_PROVIDER.equals(provider)) {
		// location 是网络定位结果
	}
	
## TencentLocation.getXXX()返回值为空？
比如, 将 `getCity()` 的返回值打印出来发现为空。

由于使用场景的差异非常大(有些app只需将坐标点显示在底图上，有些app需要街道地区，有些app需要知道行政区划，有些app需要附近的POI，等等)，定位请求有不同的 `Request Level`。

<font color="red">不同的 Level 的 TencentLocationRequest 请求得到的 TencentLocation 的完整程度不同</font>。定位前应当设置正确的 `Request Level`。

## 在新的线程中定位?
完全没有必要。<font color="red">建议直接在主线程中调用 `TencentLocationManager.requestLocationUpdates()` 来定位</font>。

原因如下：

1. 定位SDK内部有相应的工作线程用于处理网络请求/响应过程，不会阻塞主线程
2. 在主线程发起定位， `TencentLocationListener.onLocationChanged()` 是在主线程中回调，可直接操作 UI

## 一定要在新的线程中定位，如何实现?
如上所说，没必要在新的线程中定位。使用如下方式可以将定位SDK内部操作放到新的线程中，一定程度上减少主线程负担：

	TencentLocationManager locationManager = ...
	// 创建并运行 HandlerThread
	HandlerThread thread = new HandlerThread();
	thread.start();
	// 在主线程发起定位
	locationManager.requestLocationUpdates(request, listener, thread.getLooper());

## 可否连续调用 requestLocationUpdates ()?
目前只支持一个 TencentLocationListener。连续调用后，将自动取消原来的 listener，并使用新的 request 和 listener 定位。

可以利用这个特点来调整定位频率而不必 `removeUpdates()` 后再 `requestLocationUpdates()`。

定位完成后，只需要调用 **一次** `removeUpdates()` 来取消定位。

## so库引起的崩溃，怎么办？
建议通过下面步骤排查：

1. 检查设备的 CPU 架构
2. 检查是否将 so 文件拷贝到项目中相应的目录
3. 检查最终生成的 apk 中是否包含相应的 so 文件

## 模拟GPS定位不起作用
定位SDK使用一些策略过滤掉模拟的GPS位置，建议使用真实GPS位置测试。

