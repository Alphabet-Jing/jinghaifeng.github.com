---
layout: post
keywords: blog
description: blog
title: "被隐藏的 Android API"
categories: [Android]
tags: [Android]
---
{% include JB/setup %}

#被隐藏的 Android API

##官方 API 和隐藏 API

SDK 文档中的所有类、接口、方法以及常量都属于**官方 API** 。

Android SDK 中包含一个 JAR 文件（ android.jar ），在编译代码时会引用到它。不过它里面全是空的类，方法中的所有代码都被移除，只声明了 public 和 protected 的类。

Android 会自动隐藏某些 API ，而不需要使用 @hide 注解。这些 API 位于 **com.android.interal **包中，不属于 android.jar 文件，但确实包含了大量供 Android 平台使用的内部代码。同时，Android 系统应用还包含一些其他**隐藏的  API** ， 这些 API 通常没有提供在官方 SDK 中的 系统 ContentProvider 信息。

##发现隐藏 API 

寻找隐藏 API 最简单的方法是在 Android 源码中搜索，幸好有几个网站已对这些代码进行了索引，并提供了搜索功能。

* [androidxref.com](http://androidxref.com)
* [android 官方 reference](http://developer.android.com/reference/packages.html)

##安全调用隐藏 API

1. 常量字段

	比如广播的 action 或者 ContentProvider 的 Uri ，是实用隐藏 API 的主要部分。开发者可以直接将这些字段复制到自己的代码中使用。常用的方法为：复制源码到项目中，如果在不同 Android API 级别中被修改过，开发者需要保持各个副本，如此还能支持多种版本。

2. 需要编译时链接的 API 
	
	比如接口、类及方法，开发者有两种方法处理：
	
	*   修改 SDK 的 JAR 文件，使之包含所有需要的类和接口，并使用该 SDK 来编译应用程序。**优点**：能有效的把代码绑定到使用的设备上，无性能损失。**缺点**：处理须小心，可能引起程序崩溃。
	* 使用 Java 反射的 API 来动态查看调用的类和方法。**优点**：同时支持多个 Android 版本。**缺点**：需要查找类和方法，性能有影响。
	* 使用 Android Studio 关联源码查看（博主添加）

 	两种方法各有优缺点，需根据不同情况选择。

##隐藏 API 示例

####接收和阅读 SMS

Android 中使用隐藏 API 最常见的例子是接收和阅读 SMS 。虽然官方 API 包含了 RECEIVE_SMS 和 READ_SMS 这两个权限，但实际执行的 API 却是隐藏的。

应用程序想要接收 SMS 必须声明使用 RECEIVE_SMS 权限，并且同时实现 BroadcastReceiver，以处理收到的短信。
	
	//ContentProvider 隐藏的Uri，直接复制即可
	public static String SMS_RECEIVED_ACTION = "android.provider.Telephony.SMS_RECEIVED";
	
	//通过 "pdus" 获取 SMS 数据的隐藏键
	Object[] messages = (Object[]) intent.getSerializableExtra("pdus");
	for(Object message : messages){
	byte[] messageData = (byte[]) message;
	SmsMessage smsMessage = SmsMessage.createFromPdu(messageData);
	//处理SmsMessage

####Wi-Fi 网络共享

Android 智能手机可以启用 Wi-Fi 网络共享，以创建一个移动 Wi-Fi 热点，让其他设备链接网络。很方便的功能，同时也为开发者带来了一些问题。

当用户启用 Wi-Fi 网络共享时，Wi-Fi 状态既不是打开，也不是关闭，如果通过 API （WifiManager.getWifiState()）  查询，会得到 “未知” 的结果（博主记录：经过尝试，在4.4.2版本上返回的状态是 WIFI_STATE_DISABLED）。而 isWifiApEnabled() 是被隐藏的方法。通过反射使用该方法，可以判断正确的状态。

通过WifiManager.getConfiguredNetworks() 枚举出来的 WifiConfiguration 对象，其 preSharedKey 都被设置成了 null 。如果通过**反射**获取，则会发现preSharedKey 呈现明文密码。

####隐藏设置

Android 设备中有数百种设置，都可以通过 Settings 类来访问。除去每个设置的访问值，Android 还提供了一系列的 Intent 操作，使用它们可以操作特定的UI。

Settings 类包含一些隐藏的设置键和 Intent 操作，但应用需要弄清楚设备的细节或者呈现一个特定的设置的快捷方式时，其中的常量值是非常有用的。

####小结

在 Android 平台有大量的隐藏 API ，他们有的不仅隐藏还需要 signature 或者 system 权限保护 ，所以很多是开发者无法使用的。然而，如果创建自定义的固件，使用这些方法可以非常有效的构建访问系统 API 的高级应用。


###总结自[Android 编程实战](http://www.amazon.cn/Android%E7%BC%96%E7%A8%8B%E5%AE%9E%E6%88%98-%E8%B5%AB%E5%B0%94%E6%9B%BC/dp/B00LE6G0UI/ref=sr_1_1?ie=UTF8&qid=1437700320&sr=8-1&keywords=android+%E7%BC%96%E7%A8%8B%E5%AE%9E%E6%88%98)


