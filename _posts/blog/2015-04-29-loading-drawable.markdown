---
layout: post
keywords: blog
description: blog
title: "[开源]实现顺滑过渡动画的LoadingView"
categories: [Android]
tags: [Android,UI]
---
{% include JB/setup %}

##Screen Shot
![shot](http://7xiqgb.com1.z0.glb.clouddn.com/loadingdrawable.gif)

##[GitHub地址](https://github.com/JingHaifeng/LoadingDrawable)

##Useage
`LoadingState`

	public enum LoadingState {
        LOADING, ERROR, SUCCESS
    }
	
	
`init`

	LoadingDrawable drawable = new LoadingDrawable(this);
	drawable.setBorder();
	drawable.setMaxSweepAngle();
	drawable.setMinSweepAngle();
	
`setState`

	drawable.setLoadingState();

