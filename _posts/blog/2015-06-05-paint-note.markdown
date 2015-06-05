---
layout: post
keywords: blog
description: blog
title: "[转]Paint笔记"
categories: [Android]
tags: [Android,UI]
---

#Paint笔记

---

###构造

Paint具有三个构造

`Paint()`创建一个对象

`Paint(int flags)`创建对象时，传入定义好的属性，例如`Paint.ANTI_ALIAS_FLAG`用于抗锯齿。

`Paint(Paint paint)`使用已有的Paint对象的属性创建新的Paint，
 
	private void initPaint() {
        // 构建Paint时直接加上去锯齿属性
        mColorPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        // 直接使用mColorPaint的属性构建mBitmapPaint
        mBitmapPaint = new Paint(mColorPaint);
    }

---

###常用方法

`setARGB(int a, int r, int g, int b)`用于设置画笔的颜色，A（alpha)，R(Red)，G（Green)，B(Blue)。色值采用16进制，[0,255]之间，0x00即为完全没有，0xff为满值。

`setAlpha(int a)`用于设置Paint的透明度。

`setColor(int color)`用于设置颜色。例如`COlor.BLACK`。

`setColorFilter(ColorFilter filter)`设置颜色过滤器，可以过滤掉对应色值，比如去掉照片颜色，生成老照片效果。

`ColorFilter`有以下几个子类:
`ColorMatrixColorFilter`、`LightingColorFilter`、`PorterDuffColorFilter`

* `ColorMatrixColorFilter`通过把颜色矩阵，使对象中的色值进行变化。
	
	颜色矩阵以一组一维数组`m=[a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t]`进行存储。
	形成M矩阵：![Matrix](http://img.blog.csdn.net/20150318171652260?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
	
	图像的颜色取决于图像的RGBA值，用C矩阵来存储:![C](http://img.blog.csdn.net/20150318172047306?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
	
	两者的乘积形成新的显示矩阵：![new](http://img.blog.csdn.net/20150318172136104?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
	
	通常，改变色彩分量时，可以通过修改第五列的颜色偏量来实现:![](http://img.blog.csdn.net/20150318173529060?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
	
	或者修改对应颜色的系数来达到修改颜色的目的：![](http://img.blog.csdn.net/20150318173818779?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
	
	利用`ColorFilter`和`ColorMatrixFilter`类，通过`Paint`的`setColorFilter`方法就能改变图片展示的效果，黑白老照片、泛黄旧照片等特效可以由此得到。
	
* `LightingColorFilter`
   
        /**
     	* Create a colorfilter that multiplies the RGB channels by one color,	    * and then adds a second color,
     	* pinning the result for each component to [0..255]. The alpha components 		* of the mul and add arguments
     	* are ignored.
     	*/
       	public LightingColorFilter(int mul, int add) {
        	native_instance = native_CreateLightingFilter(mul, add);
        	nativeColorFilter = nCreateLightingFilter(native_instance, mul, add);
    	}
  
  
  `mul`和`add`分别是16进制的色值。例如`mul=0xff00ffff`、`add=0x000000ff`，将过滤红色，增强蓝色。
  `最终颜色=(mul*原色值+add)%255`。
  
* `PorterDuffColorFilter`可以用于PS，图层混合模式

        /**
     	* Create a colorfilter that uses the specified color and porter-duff 		mode.
     	*
     	* @param srcColor       The source color used with the specified
     	*                       porter-duff mode
     	* @param mode           The porter-duff mode that is applied
     	*/
    	public PorterDuffColorFilter(int srcColor, PorterDuff.Mode mode) {
        	native_instance = native_CreatePorterDuffFilter(srcColor, 			mode.nativeInt);
        	nativeColorFilter = nCreatePorterDuffFilter(native_instance, srcColor, 			mode.nativeInt);
    	}
    
  
  使用一个色值和绘制的图片进行叠加，然后选择叠加模式(PorterDuff.Mode)，PorterDuff.Mode以枚举的形式定义：
  
  
      	// these value must match their native equivalents. See SkPorterDuff.h
   			 public enum Mode {
        /** [0, 0] */
        CLEAR       (0),
        /** [Sa, Sc] */
        SRC         (1),
        /** [Da, Dc] */
        DST         (2),
        /** [Sa + (1 - Sa)*Da, Rc = Sc + (1 - Sa)*Dc] */
        SRC_OVER    (3),
        /** [Sa + (1 - Sa)*Da, Rc = Dc + (1 - Da)*Sc] */
        DST_OVER    (4),
        /** [Sa * Da, Sc * Da] */
        SRC_IN      (5),
        /** [Sa * Da, Sa * Dc] */
        DST_IN      (6),
        /** [Sa * (1 - Da), Sc * (1 - Da)] */
        SRC_OUT     (7),
        /** [Da * (1 - Sa), Dc * (1 - Sa)] */
        DST_OUT     (8),
        /** [Da, Sc * Da + (1 - Sa) * Dc] */
        SRC_ATOP    (9),
        /** [Sa, Sa * Dc + Sc * (1 - Da)] */
        DST_ATOP    (10),
        /** [Sa + Da - 2 * Sa * Da, Sc * (1 - Da) + (1 - Sa) * Dc] */
        XOR         (11),
        /** [Sa + Da - Sa*Da,
             Sc*(1 - Da) + Dc*(1 - Sa) + min(Sc, Dc)] */
        DARKEN      (12),
        /** [Sa + Da - Sa*Da,
             Sc*(1 - Da) + Dc*(1 - Sa) + max(Sc, Dc)] */
        LIGHTEN     (13),
        /** [Sa * Da, Sc * Dc] */
        MULTIPLY    (14),
        /** [Sa + Da - Sa * Da, Sc + Dc - Sc * Dc] */
        SCREEN      (15),
        /** Saturate(S + D) */
        ADD         (16),
        OVERLAY     (17);
  
  `Sa`代表Source alpha（透明度——颜色的透明度），`Da`代表Destination alpha（目标 alpha），`Sc`代表Source Color（源颜色），`Dc`代表Destination Color（目标色）。
  
  `mBitDuffPaint.setColorFilter(new PorterDuffColorFilter(DUFF_COLOR, PorterDuff.Mode.DARKEN))`变暗。
  
  `mBitDuffPaint.setColorFilter(new PorterDuffColorFilter(DUFF_COLOR, PorterDuff.Mode.LIGHTEN))`变亮。
	
	模式多尝试几次就能明白其中用法。
	
`setDither(boolean dither)`防抖动。主要在绘制渐变色彩或含有渐变的图片时，android对不含alpha通道的图片会进行转化，成为`RGB565`格式。这种格式占用内存小，但因此会出现`色带`，让人感觉过度不柔和。android针对这个问题，提出了`防抖动`，将原始颜色的过滤处进行了一些改变，从而使颜色过度更加`柔和`，让人觉得是平滑的过度。

`setFilterBitmap(boolean filter)`若设置成`ture`，图像在动画进行中，会过滤掉对bitmap图像的优化操作，加速显示速度。

`setFlags(int flags)`可以用来给Paint设置定好的属性，例如抗锯齿、防抖动等。

`setMaskFilter(MaskFilter maskFilter)`设置绘制图片边缘效果，可以模糊和浮雕。MaskFilter类含有以下几类

* `BlurMaskFilter`指定一个模糊的样式和半径来处理Paint的边缘。
* `EmbossMaskFilter`指定一个光源的方向和环境光强度来添加浮雕效果。

`setPathEffect(PathEffect effect)`用来控制绘制轮廓(线条)的方式。PathEffect类本身没有什么特殊处理，其子类比较常用。

* `CornerPathEffect`将Path的各个连接线段之间的夹角，用一种更平滑的方式连接，类似于圆弧与切线的效果。一般使用`CornerPathEffect(float radius)`来指定一个具体的圆弧半径实例化`CornerPathEffect`
* `DashPathEffect`将Path虚线化。构造函数为`DashPathEffect(float intervals[], float phase)`intervals为虚线的ON和OFF数组，其长度必须大于2，phase为绘制时的偏移量。
* `DiscretePathEffect`打散Path的线段，使得路径在原有的基础上出现打散的效果。`DiscretePathEffect(float segmentLength, float deviation)`segmentLength指定最大的段长，deviation指定偏移量。
* `PathDashPathEffect`使用图形来填充当前路径。`PathDashPathEffect(Path shape, float advance, float phase, Style style)`shape指填充图形，addvance指每个图形间的间隔，phase为绘制时的偏移量，style是该类自由的枚举，有三种:


		public enum Style {
        	TRANSLATE(0),   //!< translate the shape to each position
   					       //图形会以位置平移的方式与下一段相连接
    		ROTATE(1),      //!< rotate the shape about its center 
              			//线段连接处的图形转换以旋转到与下一段移动方向相一致的角度进行转转
        	MORPH(2);       //!< transform each point, and turn lines into curves
        					//图形会以发生拉伸或压缩等变形的情况与下一段相连接
        	Style(int value) {
          		native_style = value;
        		}
        	int native_style;
    	}
 	
* `ComposePathEffect`将两个PathEffect构成一个实例。`ComposePathEffect(PathEffect outerpe, PathEffect innerpe)`首先将innerpe表现出来，再次基础上再增加outerpe。
* `SumPathEffect`叠加效果。`SumPathEffect(PathEffect first, PathEffect secon)`分别对独立的参数效果，独立进行表现，然后将两者简单的显示在一起。与`ComposePathEffect`不同。

***phase***参数：其值不停变化，所绘制的图形也跟随偏移量进行变化，视觉效果像自己动的线。

* `setShader(Shader shader)`设置图像效果，利用Shader可以绘制出各种渐变效果。Shaderz一下有五个子类可用。

  `BitmapShader`位图图像渲染。
  
  `LinearGradient`线性渲染。
  
  `RadialGradient`环形渲染。
  
  `SweepGradient`扫描渐变渲染、梯度渲染。
  
  `ComposeGradient`组合渲染，可以将其他子类组合在一起。
  
  `LinearGradient`、`RadialGradient`、`SweepGradient`均是将颜色进行处理，形成柔和的过度，即渐变。`BitmapShader`则是直接使用位图进行渲染，类似于贴图。在贴图过程中需要选择相应的模式，分别为以下三种：
  
  
      public enum TileMode {
        /**
         * replicate the edge color if the shader draws outside of its
         * original bounds
         * 边缘拉伸，使用边缘的最后一个像素拉伸。
         */
        CLAMP   (0),
        /**
         * repeat the shader's image horizontally and vertically
         * 在水平方向或者垂直方向重复摆放，两个相邻的图像有间隙。
         */
        REPEAT  (1),
        /**
         * repeat the shader's image horizontally and vertically, alternating
         * mirror images so that adjacent images always seam
         * 在水平或者垂直方向交替镜像，两个相邻图像没有间隙，像镜子一样。
         */
        MIRROR  (2);
    
        TileMode(int nativeInt) {
            this.nativeInt = nativeInt;
        }
        final int nativeInt;
    }
  
  1. `linearGradient`（线性渲染——线性渐变)
  
	
		 /** Create a shader that draws a linear gradient along a line.
         @param x0           The x-coordinate for the start of the gradient line
         @param y0           The y-coordinate for the start of the gradient line
         @param x1           The x-coordinate for the end of the gradient line
         @param y1           The y-coordinate for the end of the gradient line
         @param  colors      The colors to be distributed along the gradient line
         @param  positions   May be null. The relative positions [0..1] of
                            each corresponding color in the colors array. If this is null,
                            the the colors are distributed evenly along the gradient line.
         @param  tile        The Shader tiling mode
    	  */
   		  public LinearGradient(float x0, float y0, float x1, float y1, int colors[], float positions[],
            TileMode tile) {
        	if (colors.length < 2) {
            	throw new IllegalArgumentException("needs >= 2 number of colors");
        	}
        	if (positions != null && colors.length != positions.length) {
            	throw new IllegalArgumentException("color and position arrays must be of equal length");
        	}
        	mType = TYPE_COLORS_AND_POSITIONS;
        	mX0 = x0;
        	mY0 = y0;
        	mX1 = x1;
        	mY1 = y1;
        	mColors = colors;
	    	mPositions = positions;
       		mTileMode = tile;
        	init(nativeCreate1(x0, y0, x1, y1, colors, positions, tile.nativeInt));
    		}
	 
  	确定起始点(x0,y0)、终点(x1,y1)，颜色值colors[]，颜色位置positions[]——确定各个颜色值纯色点所在的位置，模式title(Shader.Mode.XXX)

  	
  2. `RadialGradient`(环形渲染)
  
       
		 /** Create a shader that draws a radial gradient given the center and radius.
        	@param centerX  The x-coordinate of the center of the radius
       		@param centerY  The y-coordinate of the center of the radius
        	@param radius   Must be positive. The radius of the circle for this gradient.
        	@param colors   The colors to be distributed between the center and edge of the circle
        	@param stops    May be <code>null</code>. Valid values are between <code>0.0f</code> and
                        <code>1.0f</code>. The relative position of each corresponding color in
                        the colors array. If <code>null</code>, colors are distributed evenly
                        between the center and edge of the circle.
        	@param tileMode The Shader tiling mode
    		*/
    		public RadialGradient(float centerX, float centerY, float radius,
               	@NonNull int colors[], @Nullable float stops[], @NonNull TileMode 				tileMode) {
        		if (radius <= 0) {
            		throw new IllegalArgumentException("radius must be > 0");
        		}
        	if (colors.length < 2) {
            	throw new IllegalArgumentException("needs >= 2 number of colors");
        	}
	        if (stops != null && colors.length != stops.length) {
    	        throw new IllegalArgumentException("color and position arrays must be of equal length");
        	}
        	mType = TYPE_COLORS_AND_POSITIONS;
	        mX = centerX;
    	    mY = centerY;
        	mRadius = radius;
	        mColors = colors;
    	    mPositions = stops;
        	mTileMode = tileMode;
	        init(nativeCreate1(centerX, centerY, radius, colors, stops, tileMode.nativeInt));
    		}


	  和`LinearGradient`相似，以(centerX,centerY)为起点，radius为半径，颜色值colors,颜色值对应位置stops[]，模式mode。
	  
  3. `SweepGradient`扫描渐变、梯度渐变
  
    
		 /**
     	 * A subclass of Shader that draws a sweep gradient around a center point.
     	 *
    	 * @param cx       The x-coordinate of the center
     	 * @param cy       The y-coordinate of the center
     	 * @param colors   The colors to be distributed between around the center.
     	 *                 There must be at least 2 colors in the array.
     	 * @param positions May be NULL. The relative position of
     	 *                 each corresponding color in the colors array, beginning
     	 *                 with 0 and ending with 1.0. If the values are not
     	 *                 monotonic, the drawing may produce unexpected results.
     	 *                 If positions is NULL, then the colors are automatically
     	 *                 spaced evenly.
     	 */
    	 public SweepGradient(float cx, float cy,
                         int colors[], float positions[]) {
        	if (colors.length < 2) {
         	   throw new IllegalArgumentException("needs >= 2 number of colors");
        	}
        	if (positions != null && colors.length != positions.length) {
            	throw new IllegalArgumentException(
                        "color and position arrays must be of equal length");
        	}
        	mType = TYPE_COLORS_AND_POSITIONS;
        	mCx = cx;
        	mCy = cy;
        	mColors = colors;
        	mPositions = positions;
        	init(nativeCreate1(cx, cy, colors, positions));
    	 }
     
     和之前的参数一致,用法也相似。
     
  4. `BitmapShader`位图渲染
  
     
         /**
     	   * Call this to create a new shader that will draw with a bitmap.
     	   *
           * @param bitmap            The bitmap to use inside the shader
           * @param tileX             The tiling mode for x to draw the bitmap in.
           * @param tileY             The tiling mode for y to draw the bitmap in.
           */
    		public BitmapShader(Bitmap bitmap, TileMode tileX, TileMode tileY) {
        	mBitmap = bitmap;
        	mTileX = tileX;
        	mTileY = tileY;
        	final long b = bitmap.ni();
        	init(nativeCreate(b, tileX.nativeInt, tileY.nativeInt));
    		}
     
     
     例如：将![block](http://img.blog.csdn.net/20150322124138698?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)横向纵向铺满全屏形成![wall](http://img.blog.csdn.net/20150322124217848?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
     
     可以减少包大小，运行时内存消耗，等很多效果。
     
  5. `ComposeShader`组合渲染
  
    
     
         /** Create a new compose shader, given shaders A, B, and a combining mode.
         When the mode is applied, it will be given the result from shader A as its
         "dst", and the result from shader B as its "src".
         @param shaderA  The colors from this shader are seen as the "dst" by the mode
         @param shaderB  The colors from this shader are seen as the "src" by the mode
         @param mode     The mode that combines the colors from the two shaders. If mode
                        is null, then SRC_OVER is assumed.
    	 */
    	 public ComposeShader(Shader shaderA, Shader shaderB, Xfermode mode) {
         	mType = TYPE_XFERMODE;
         	mShaderA = shaderA;
         	mShaderB = shaderB;
         	mXferMode = mode;
         	init(nativeCreate1(shaderA.getNativeInstance(), shaderB.getNativeInstance(),(mode != null) ? mode.native_instance : 0));
    	 }
     
     通过PorterDuff.Mode（叠加模式）或者Xfermode（混合模式）进行渲染
 
* `setShadowLayer(float radius,float dx,float dy,int color)`在图形的下面设置阴影层，产生阴影效果，radius为阴影的角度，dy、dy为阴影在x、轴的距离，color为阴影的颜色。
* `setStyle(Paint.Style style)`设置画笔的模式，FILL、STROKE或FILL_OR_STROKE。
* `setStokeCap(Paint.Cap cap)`设置画笔在FILL、FILL_OR_STROKE时，画笔的样式Cap.ROUND（圆形）或Cap.SQUARE(方形)。
* `setStrokeJoin(Paint.Join join)`设置图形结合的方式，如平滑等。
* `setStokeWidth(float width)`设置画笔在FILL、FILL_OR_STOKE时，画笔的粗细程度。
* `setXfermode(Xfermode xfermode)`设置图形重叠的处理方式，合并、交集或并集。

###Text文本绘制相关
* `setFakeBoldText(boolean fakeBoldText)`模拟实现粗体文字，小字体上效果很差。
* `setSubpixelText(boolean subpixelText)`ture时，保证在绘制斜线时候使用抗锯齿效果，来平滑斜线的外观。
* `setTextAlign(Paint paint)`设置文字的对齐方向
* `setTextScaleX(float scaleX)`设置绘制文字x轴的缩放比例，实现文字拉伸效果。
* `setTextSize(float textSize)`设置文字字号大小，默认单位为SP。
* `setTextSkewX(float skewX)`设置斜体文字，skewX为倾斜度。
* `setTypeface(Typeface typeface)`设置Typeface对象，即字体风格，包括粗体，斜体、衬线体或非衬线体。
* `setUnderlineText(boolean underlineText)`设置下划线的文字效果。
* `setStrikeThruText(boolean strikeThruText)`设置带有删除线的效果。

####引用来自[Ajian_studio](http://blog.csdn.net/tianjian4592/article/details/44336949)的博客

  	 
  

