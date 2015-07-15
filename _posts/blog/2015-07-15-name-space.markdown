---
layout: post
keywords: blog
description: blog
title: "开发命名规范简略记录"
categories: [Android]
tags: [Android]
---
{% include JB/setup %}

#开发命名规范简略记录

1.  包命名：
	使用全小写，构成为：机构名称.App名称.库名。
	
		package com.mymoney.sms.ui;

2. 类、接口命名：尽量用清晰明确的英文单词给类命名，如果需要多个单词，单词之间不需要连接字符（比如下滑线），并且每个单词首字母大写。

		public class View implements Drawable.Callback, Drawable.Callback2, KeyEvent.Callback,
        AccessibilityEventSource {

3. 方法命名：方法命名和类的命名也类似，其区别是第一个单词的首字母小写。
	
		protected void onResume() {

		}
	
4. 类中成员变量命名：

	* `android`平台相关 非**static** 、**public**修饰的成员变量，以m开头，后面接表明变量意义的名称，每个单词首字母大写。`get`、`set`方法不带m开头。
		
			private RelativeLayoute mUserRlyt;
			
	*  非`android`平台相关业务层`java`代码，成员变量遵循`java`命名规则。
			
			private int number;

	* 非 **final** 修饰的 **static** 成员变量，以s开头，之后每个单词首字母大写。
	
			private static List sAccountList;

	* 使用** static final** 修饰的成员变量，全大写。

			public static final String USER_NAME = "user";

	* 一般 **public** 变量不带特殊字符开头，第一个单词首字母小写，后续单词首字母大写 ，`建议不要使用 public 修饰的非 static 成员变量`。

	* 修饰符排列顺序：
			
			public/protected/private  abstract  static  final  transient  volatile   synchronized  native

5. 临时变量命名：
	尽量使用意义清晰明确的单词命名，第一个单词首字母小写，其余首字母大写。单词与单词之间不需要额外的连接符。

			final int actionMasked = action & MotionEvent.ACTION_MASK;

6. 简写命名：
	对于HTML、URL、PWD等简写，当其作为类名，函数名，变量名使用时，作为一个单词对待。
	
	* 作为类名时，首字母大写，其余小写。
	* 作为函数名和变量名时，放在其他单词后面一律首字母大写，其余小写；若在其他单词前面一律小写。

			URL url;
			String urlString;

7. 异常规范：
	不要catch异常而不做任何处理，可让异常抛出，但最好生成一个新的具体的异常抛出，并记录错误信息。
	
	值得**注意**：NullPointerException 原则上不允许catch；OutOfMenoryError在解码图片时或者经常出现内存紧张的字符串拼接时，可catch，其它地方尽量不要catch。

8. 锁与同步
	* 若不是特殊需求，尽量使用java的原生同步原语`synchronized`而不是使用重入锁`ReentrantLock`。但是，如果在获取锁失败后不打算等待的话，可以使用`ReentrantLock`。
	* 不要在主线程中做`IO`等耗时操作，主线程只用于用户界面，主线程被卡5秒Android系统将会ANR。当使用重入锁时，使用try-finally语句，在finally中释放。

9. 类属性、方法摆放位置：
	除非某些变量有业务聚合属性，可单独抽离放置，否则同一访问级别的都放在一起。

		格式：
		类中的成员变量在类文件中，按照以下顺序定义
		static final 变量
		static	变量
		Public 类型常量
		Public 类型变量
		Protected 类型变量
		Package 类型变量（默认不写Package）
		Private  类型变量

		类中的方法在类文件中，按照以下顺序定义
		static 方法
		Public 类型方法
		Protected 类型方法
		Package 类型方法（默认不写Package）
		Private  类型方法

10. 调用、使用：
	* 对于类成员变量的访问，需要`set`、`get`方法完成。
	* 对于类变量和方法，避免使用一个对象访问一个类的静态变量和方法，应该用类名代替。
	* 匿名类：如果时一次性使用的匿名类，可以在使用时直接new，如果可能服用，先初始化到成员变量中存储。
	* 内部类：若其中有需要公开给外部使用的数据或方法，并有一定逻辑，需将其改写成独立类。
	
11. 文件编码：
	* 源文件统一使用`utf-8`
	* 非图片形式的资源文件，编码也用`utf-8`。

12. Androids资源名称约定：
	资源文件夹是扁平化的结构，以业务名称作为前缀，把相关资源聚合在一起，公共资源以`common`开头，如：`common_dialog_bg`形式命名。
	* drawable:

			业务名称_btn:  按钮资源前缀,比如 xml定义名称是add_transaction_btn 实际引用的图片是add_transaction_btn_pressed和add_transaction_btn_normal ,如果有checked状态则add_transaction_btn_checked
			业务名称_bg：背景资源前缀，比如 transfer_cost_in_new_bg
			业务名称_icon：图标前缀，比如 kaniu_intro_icon/category_icon_qtsz
	
	* layout:
		
			业务名称_activity：activity后缀
			业务名称_父类控件名称：自定义控件后缀,ColorButton 继承于Button 
			common_xxx_dialog：对话框布局后缀
			业务名称_popup：PopupWindow后缀
			业务名称_fragment：Fragment后缀
			业务名称_listview_item：列表类视图某一元素布局后

			layout xml 的命名必须以 全部单词小写，单词间以下划线分割，并且使用名词或名词词组，即使用 模块名_功能名称,来命名

	* id:

			layout 中所使用的id必须以全部单词小写，单词间以下划线分割，并且使用名词或名词词组+控件名称缩写，并且要求能够通过id直接理解当前组件要实现的功能。 

			常用缩写，详细参考工程原有代码: TextView >tv  Button>btn  ListView>lv  LinearLayout>ly  

			范例:
			错误：某TextView @+id/accountName 
			正确:  @+id/account_name_tv
			错误：某Button@+id/addTransactionButton 
			正确: @+id/add_transaction_btn

	* anim:
		
			anim以效果命名，如果是成对出现的动画，应该能够在名字中看出对应关系。如：fade_in.xml, fade_out.xml

	* AndroidManifest.xml:

			AndroidManifest中的四大组件需要有简要的功能描述；权限需要说明使用原因；其它自定义的部分，比如meta-data，需说明是哪个模块使用的

	* style:

			布局中的可以抽取的样式一定要抽取,属性的排列方式要遵循一定的自然顺序:比如先 android:layout_开头的布局宽高 ,然后是文字相关 android:gravity  android:textColor，最后是背景.

			样式常识：所有的TextView要显示定义强制单行和截断方式（防止文字过长撑坏布局）、字体颜色（不同的rom或主题默认的颜色不一）。
	
	* 其他：
			
			color/arrays等资源定义基本遵循以业务名称_描述用途的方式命名.

			strings字符串Key取值以：业务模块_字符描述命名（其中，字符描述单词之间用_分割）
			布局中的控件ID，以view类型缩写_模块_用途 命名。


13. Android 相关代码命名：
	为使业务关联的文件显示在一起，代码采用后缀的形式加以区分：
	* Activity 以 Activity 结尾
	* Fragment 以 Fragment 结尾
	* 自定义View 以 View 结尾
	* 自定义Dialog 以Dialog结尾
	* ListView的 Item视图，以 ItemView结尾
	


			
