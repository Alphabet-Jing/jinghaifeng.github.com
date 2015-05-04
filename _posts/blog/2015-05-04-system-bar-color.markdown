---
layout: post
keywords: blog
description: blog
title: "改变 Android System bar 颜色"
categories: [Android]
tags: [Android,UI]

---
{% include JB/setup %}

##SnapShot
![](http://7xiqgb.com1.z0.glb.clouddn.com/systembarcolor.png)

---

###实现方式
 * Android 4.4以上，使用SystemBarTintManager 
 * Android 5.0,使用android:statusBarColor，Window.setStatusBarColor 
 * 使用第三方兼容库 SystemBarTint
 
 推荐最后一种方式
 
 ---
 
###[SystemBarTint](https://github.com/jgilfelt/SystemBarTint)简单使用
`gradle引用：`
	
    compile 'com.readystatesoftware.systembartint:systembartint:1.0.3'

`Java Code 设置：`

	
    private void initSystemBar() {

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            setTranslucentStatus(true);
        }

        SystemBarTintManager tintManager = new SystemBarTintManager(this);
        tintManager.setStatusBarTintEnabled(true);
        tintManager.setStatusBarTintResource(R.color.material_deep_teal_500);
    }
    
 `XML设置：`
 
 	
    android:fitsSystemWindows="true"
    

	
	