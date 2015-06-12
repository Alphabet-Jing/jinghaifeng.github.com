---
layout: post
keywords: blog
description: blog
title: "Canvas笔记"
categories: [Android]
tags: [Android]
---
{% include JB/setup %}

##Canvas笔记

`Cavas`中的方法分为如下几类：

 1. `save`、`restore` 等与层的保存与回滚。
 2. `scale`、`rotate`、`clipXXX` 等对画布进行操作的方法。
 3. `drawXXX` 等一些列绘画相关的方法。

###drawXXX

####`drawBitmap`

其中有如下方法：

![drawbitmap](http://img.blog.csdn.net/20150413201326694?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

`drawBitmap(Bitmap,Rect,Rect,Paint)`

第一个Rect代表要绘制的bitmap的区域，第二个Rect代表是要讲bitmap绘制到屏幕的位置。即，`SrcRect`、`DesRect`。

动态修改`DesRect`，利用`valueAnimator`。能实现顺滑的动画。

####`canvas.translate()`画布的平移

	/**
     * Preconcat the current matrix with the specified translation
     *
     * @param dx The distance to translate in X
     * @param dy The distance to translate in Y
    */
    public void translate(float dx, float dy) {
        native_translate(mNativeCanvasWrapper, dx, dy);
    }
   
值得注意的是，画布的每次移动都是叠加的，例如:连续两次`canvas.translate(100, 100)`，将会画布移动的差为(200,200)。

	@Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.drawColor(Color.BLUE);
        canvas.translate(100, 100);
        canvas.drawRect(new Rect(0, 0, 400, 400), mPaint);
        canvas.translate(100, 100);
        canvas.drawRect(new Rect(0, 0, 400, 400), mPaint);
    }
    
![](http://img.blog.csdn.net/20150506125132147?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

例如这个特性，可以绘制规律图形时会很方便,比如尺子。

![chi](http://img.blog.csdn.net/20150506130806228?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

####`canvas.scale()`画布的缩放

    /**
     * Preconcat the current matrix with the specified scale.
     *
     * @param sx The amount to scale in X
     * @param sy The amount to scale in Y
     */
    public void scale(float sx, float sy) {
        native_scale(mNativeCanvasWrapper, sx, sy);
    }

    /**
     * Preconcat the current matrix with the specified scale.
     *
     * @param sx The amount to scale in X
     * @param sy The amount to scale in Y
     * @param px The x-coord for the pivot point (unchanged by the scale)
     * @param py The y-coord for the pivot point (unchanged by the scale)
     */
    public final void scale(float sx, float sy, float px, float py) {
        translate(px, py);
        scale(sx, sy);
        translate(-px, -py);
    }
    
canvas缩放提供了2个方法。第一个方法默认以(0,0)为中心，sx、sy为缩放比例。第二个方法以(px,py)为中心缩放。从源码上可看出，canvas实际上是进行两次位移，一次缩放的组合。

####`canvas.rotate()`画布的旋转


    /**
     * Preconcat the current matrix with the specified rotation.
     *
     * @param degrees The amount to rotate, in degrees
     */
    public void rotate(float degrees) {
        native_rotate(mNativeCanvasWrapper, degrees);
    }

    /**
     * Preconcat the current matrix with the specified rotation.
     *
     * @param degrees The amount to rotate, in degrees
     * @param px The x-coord for the pivot point (unchanged by the rotation)
     * @param py The y-coord for the pivot point (unchanged by the rotation)
     */
    public final void rotate(float degrees, float px, float py) {
        translate(px, py);
        rotate(degrees);
        translate(-px, -py);
    }
    
与`scale`相似，旋转会按照设置的中心点来进行变化。一样是利用位移来设置中心点。利用旋转可以绘制出很规整的图形,如：

![clock](http://img.blog.csdn.net/20150507130524302?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

####`canvas.skew()`画布的斜切

    /**
     * Preconcat the current matrix with the specified skew.
     *
     * @param sx The amount to skew in X
     * @param sy The amount to skew in Y
     */
    public void skew(float sx, float sy) {
        native_skew(mNativeCanvasWrapper, sx, sy);
    }

其中sx,为画布在x轴上的tan值；sy为画布y轴上的tan值。

`canvas.skew(1, 0)`![](http://img.blog.csdn.net/20150507132403246?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

`canvas.skew(0,1)`![](http://img.blog.csdn.net/20150507132922207?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

####引用来自[Ajian_studio](http://blog.csdn.net/tianjian4592/article/details/44336949)的博客