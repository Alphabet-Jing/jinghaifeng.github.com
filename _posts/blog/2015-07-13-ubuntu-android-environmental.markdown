---
layout: post
keywords: blog
description: blog
title: "Ubuntu 14.04 Android 开发环境搭建简略记录"
categories: [Android]
tags: [Android]
---
{% include JB/setup %}

#Ubuntu 14.04 Android 开发环境搭建简略记录

由于工作原因，需将开发环境迁移至 `Ubuntu` ，遂将个别细节记录下来，以备不时之需。

* 为集中管理各个软件及`SDK`，我直接建立`/home/{user}/Application`文件夹来存放所有软件。

* 同时在`/etc/profile.d/`目录下，建立`java_home.sh`、`android_home.sh`、`android_studio.sh`、`genymotion.sh`等脚本，差量地配置各软件的环境变量。好处是：可不用修改系统本身的
配置文件，增删改某一脚本时，并不会影响其他脚本(修改后需要使用`source`命令或重新登录)。

		# /etc/profile 文件中声明将会遍历profile.d中的 *.sh 文件
		if [-d /etc/profile.d]; 
		  then 
		  for i in /etc/profile.d/*.sh;	 
		  do 
		    if [-r $i]; then 
		      .$i 
		    fi 
		  done 
		  unset i 
		fi          


##JDK & ADK环境
* 获取 JDK 及 ADK 压缩包，解压至`/home/{user}/Application`。
* 通过pwd获取绝对路径（自信手打亦可...），配置 JAVA_HOME 、JRE_HOME 和 ANDROID_HOME。
* 将 ADK 中的 tools 、 platform-tools 添加至 PATH。

		# 以 android_home.sh 为例
		export ANDROID_HOME=/home/{user}/Application/{android_sdk}
		export PATH="$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools"

##Android Studio配置
* 获取 Android Studio 压缩包，解压至`/home/{user}/Application`。
* 建立`android_studio.sh`。
* 修改`idea.properties`，末尾处添加`disable.android.first.run=true`。防止因`google`被墙，导致 Android Stduio 卡在初始界面。
* 修改`studio.vmoptions`，修改内存分配，防止 Android Studio 在某些情况下出现`OutOfMemory`。

		# 初始堆内存块
		-Xms1024m 
		# 最大允许堆内存
		-Xmx2048m
		...
	
##Git 配置

* 安装命令`sudo apt-get install git` （版本较老的 Debian 或 Ubuntu 可用`sudo apt-get install git-core`）。
* 关于 git gui ，可选用`SmartGit`——与`SmartSVN`师出同门，在`OS X`的表现不错，只是不支持`BitBucket`较麻烦，需要自己设置。也可以直接使用 Android Studio 中 CVS 的 git plugin。
