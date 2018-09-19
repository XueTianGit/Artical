title: Android进阶-Activity跳转动画、状态选择器、Shape
date: 2016/2/29 15:41:12    
categories: Android
---


# Android进阶-Activity跳转动画、状态选择器、Shape #

## Point1 (自定义对话框) ##

- 一般步骤：	   
- 		1.设计对话框的UI界面（布局文件）
- 		2.使用对话框装载布局文件

> 范例（以AlerDialog为例）：
		
		AlertDialog.Builder builder = new AlertDialog.Builder(this);
		 AlertDialog dialog = builder.create();
		View view = View.inflate(this, R.layout.dialog_input_password, null);
		// dialog.setView(view);// 将自定义的布局文件设置给dialog
		dialog.setView(view, 0, 0, 0, 0);// 设置边距为0,保证在2.x的版本上运行没问题
		dialog.show();

## Pinit2（MD5的简单使用） ##

		public static String encode(String password) {
		
				MessageDigest instance = MessageDigest.getInstance("MD5");// 获取MD5算法对象
				byte[] digest = instance.digest(password.getBytes());// 对字符串加密,返回字节数组
	
				StringBuffer sb = new StringBuffer();
				for (byte b : digest) {
					int i = b & 0xff;// 获取字节的低八位有效值
					String hexString = Integer.toHexString(i);// 将整数转为16进制
					if (hexString.length() < 2) {
						hexString = "0" + hexString;// 如果是1位的话,补0
					}
	
					sb.append(hexString);
				return sb.toString();
		
		}
	- 在线破解MD5的网站： www.cmd5.com
	
##  Pinit3 （.9.png图片） ##
>一般我们将图片设置给Android中的布局文件时，图片可能会被拉伸，由于是控件随意拉伸
> 图片，图片的效果往往不会使我们满意，我们可以使用.9.png格式的图片，来使图片被拉伸时
> 按照我们的意愿拉伸出漂亮的图片。
> 我们可以使用SDk自带的软件来制作.9.png文件
> 软件位置： sdk\tools\draw9patch.bat
> 使用黑线来限制区域： Ctrl+左键：快速产生黑线  shift+左键：去除黑线
> 右、下线： 限制文字区域
> 左、上线： 限制被拉伸区域

## Point4（selector） ##
>我们经常会看到，我们在点击一个按钮时，按钮会显示不同的状态（样子），这个效果可以使用选择器来完成。

使用步骤：
	
	1. 在 drawable\目录下，创建选择器xml文件
	2. 设置被操作控件，不同情况下的不同状态
	3. 引用选择器

范例：

	<!-- 在 drawable\btn_select.xml-->
	<selector xmlns:android="http://schemas.android.com/apk/res/android" >
	    <!-- default -->
	    <item android:drawable="@drawable/function_greenbutton_normal" ></item>
	    <!-- pressed -->	
	    <item android:drawable="@drawable/function_greenbutton_pressed" android:state_pressed="true">
		</item>
	</selector>
	
	<!-- Button控件引用选择器-->
	<Btton
        android:background="@drawable/btn_green_selector" /Btton>

- Point（TextView的点击事件）
> TextView要想添加点击事件应将属性 clickble="true"


## Point5（Activity跳转动画） ##
一般步骤：

1. 首先应自定义动画的XML文件  （在 anim\ 下）
2. Activity跳转时开启动画（覆盖）

范例：

	<!--anim\skip_in.xml -->
	<translate xmlns:android="http://schemas.android.com/apk/res/android"
	    android:duration="500"
	    android:fromXDelta="100%p"
	    android:toXDelta="0" >
	</translate>
	<!--anim\skip_out.xml -->
	<translate xmlns:android="http://schemas.android.com/apk/res/android"
	    android:duration="500"
	    android:fromXDelta="0"
	    android:toXDelta="-100%p" >   
	</translate>
	- p 表示父容器（屏幕）

	//Activity跳转时开启动画
	overridePendingTransition(R.anim.skip_in, R.anim.skip_out);//进入动画和退出动画

## Point6（Shape） ##

我们可以使用Shape来绘制简单的图形。
在 drawable\下定义自定义图形的XML文件

例如：
	<shape xmlns:android="http://schemas.android.com/apk/res/android" >
	    
	   	<!-- 角度 -->
	    <corners android:radius="5dp" />
	
	    <!-- 渐变   -->
	    <gradient
	        android:centerColor="#fff"
	        android:endColor="#0f0"
	        android:startColor="#f00" />
	    <!-- 纯色 -->
	    <solid android:color="#a000" />
	    
	    <!-- 边框 -->   
	    <stroke 
	        android:width="1dp"
	        android:color="#000"
	        android:dashWidth="5dp"
	        android:dashGap="3dp"
	        />
	</shape>

定义完后，在其他地方引用即可。
