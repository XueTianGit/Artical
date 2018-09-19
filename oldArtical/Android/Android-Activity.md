title: Android-Activity
date: 2016/2/28 10:31:15 
categories: Android
---

# Android-Activity #

## 概述 ##
Activity作为Android的四大组件之一，地位不用说。Activity是一个与用记交互的系统模块，几乎所有的Activity都是和用户进行交互的，
用户可以用来交互为了完成某项任务，例如拨号、拍照、发送email、看地图。每一个activity被给予一个窗口，在上面可以绘制用户接口。
窗口通常充满屏幕，但也可以小于屏幕而浮于其它窗口之上。

## 创建Activity ##
步骤：

- 新建一个类继承Activity, 应重写onCreate()方法。

- 在清单文件中配置<activity>标签。

----------
	-> 标签中如果带有这个子节点，则会在系统中创建一个快捷图标，即此activity为入口activity。
		 <intent-filter>
             <action android:name="android.intent.action.MAIN" />
             <category android:name="android.intent.category.LAUNCHER" />
         </intent-filter>

- 新建与Activity相应的布局文件

----------
	范例 对于下面这个配置， 就会出现一个快捷图标，且名字为“主界面”：
		 <application
	        android:allowBackup="true"
	        android:icon="@drawable/ic_launcher"
	        android:label="@string/app_name"
	        android:theme="@style/AppTheme" >
	
	        <activity
	            android:name="com.suixin.createactivity.MainActivity"
	 
				android:icon="@drawable/photo2"
	            android:label="主界面" >
	
	            <intent-filter>
	                <action android:name="android.intent.action.MAIN" />
	                <category android:name="android.intent.category.LAUNCHER" />
	            </intent-filter>
	
	        </activity>  	
		
			<activity android:name="com.suixin.createactivity.SecondActivity"
			android:icon="@drawable/photo3"    
			android:label="第二个界面">
			</activity>
	    </application>

> NT：只有在配置Activity是配置了这个过滤器，才会显示快捷图标。如果整个应用都没有配置这个意图过滤器。则应用是不会显示图标的，即没有入口activity

## Activity的跳转 ##
> 分为应用间跳转和应用内跳转。

- 显示与隐式跳转：
> - Activity的跳转需要创建Intent对象，通过设置intent对象的参数指定要跳转Activity
> - 通过设置Activity的包名和类名实现跳转，称为显式意图
> - 通过指定动作实现跳转，称为隐式意图

即activity的跳转时通过Intent完成的。  ->    	startActivity(intent);

以下为几种跳转方式：

----------

	隐式跳转（必须配置了<intent-filter>）：
			//1 隐式跳转到SecondActivity
			intent.setAction("com.itheima.sa2");
	    	intent.setType("text/username");
	    	intent.setData(Uri.parse("heima2:qwe123"));
	    	//intent.setDataAndType(Uri.parse("heima2:qwe123"), "text/username");  //顶上面两句
	    	//不写的话，系统会自动添加默认的category
	    	intent.addCategory(Intent.CATEGORY_DEFAULT);
	    	startActivity(intent);
	
			//2  隐式跳转到拨号器
	    	intent.setAction(Intent.ACTION_DIAL);
	    	startActivity(intent);
			
			//3 隐式跳转到浏览器
	    	intent.setAction(Intent.ACTION_VIEW);
	    	intent.setData(Uri.parse("http://www.baidu.com"));
	    	startActivity(intent);
			
	显式跳转：
			//1
	    	intent.setClass(this, SecondActivity.class);  		//显示意图   跳转到本应用的activity并不需要制定包名和类名
			startActivity(intent);
			//2 显示跳转到拨号器
	    	intent.setClassName("com.android.dialer", "com.android.dialer.DialtactsActivity");
	    	startActivity(intent);
			//3显示跳转到浏览器
		    intent.setClassName("com.android.browser", "com.android.browser.BrowserActivity");
	    	startActivity(intent);

> 在我们想要跳转到别的应用的activity时，我们可以去看应用源码的配置文件（android源码）

SecondActivity的配置

----------

     <activity android:name=".SecondActivity">  //. 省略包名
            <intent-filter >
                <action android:name="com.itheima.sa"/>
                <action android:name="com.itheima.sa3"/>
                <data android:scheme="heima"/>
                <data android:scheme="heima3"/>
                <category android:name="android.intent.category.DEFAULT"/>
            </intent-filter>
            
            <intent-filter >
                <action android:name="com.itheima.sa2"/>
                <data android:scheme="heima2" android:mimeType="text/username"/>
                <category android:name="android.intent.category.DEFAULT"/>
            </intent-filter>
        </activity>

	对于  </intent-filter>
	action   指定动作（可以自定义，可以使用系统自带的）
	data     指定数据（操作什么内容）
	category 类别 （默认类别，机顶盒，车载电脑）
	**在隐式启动时，多个 <intent-filter >  就代表用多种启动方式。**

### 获取通过setData传递的数据
	//获取启动此Activity的intent对象
	Intent intent = getIntent();
	Uri uri = intent.getData();     //Sop(uri) -> heima2:qwe123  
###显式意图和隐式意图的应用场景
* 显式意图用于启动同一应用中的Activity
* 隐式意图用于启动不同应用中的Activity
	* 如果系统中存在多个Activity的intent-filter同时与你的intent匹配，那么系统会显示一个对话框，列出所有匹配的Activity，由用户选择启动哪一个.

### Activity跳转时传递数据 

* Activity通过Intent启动时，可以通过Intent对象携带数据到目标Activity

		Intent intent = new Intent(this, SecondActivity.class);
    	intent.putExtra("maleName", maleName);
    	intent.putExtra("femaleName", femaleName);
    	startActivity(intent);

* 在目标Activity中取出数据

		Intent intent = getIntent();
		String maleName = intent.getStringExtra("maleName");
		String femaleName = intent.getStringExtra("femaleName");

## Activity的生命周期 ##
![](http://7xrbxa.com1.z0.glb.clouddn.com/androidactivity%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F1.jpg)

生命周期的相关方法即相应的状态

- void onCreate()   Activity已经被创建完毕
- void onStart()    Activity已经显示在屏幕，但没有得到焦点
- void onResume()   Activity得到焦点，可以与用户交互
- void onPause()    Activity失去焦点，无法再与用户交互，但依然可见
- void onStop()     Activity不可见，进入后台
- void onDestroy()  Activity被销毁
- void onRestart()  Activity从不可见变成可见时会执行此方法

>- 这些方法可以帮助我们：
>- Activity创建时需要初始化资源，销毁时需要释放资源；或者播放器应用，在界面进入后台时需要自动暂停

	完整生命周期（entire lifetime）：
	
	onCreate-->onStart-->onResume-->onPause-->onStop-->onDestory
	
	可视生命周期（visible lifetime）：
	
	onStart-->onResume-->onPause-->onStop
	
	前台生命周期（foreground lifetime）：
	onResume-->onPause

## Activity的启动模式 ##

> 体现：

	<activity 
	    android:launchMode="singleInstance"
	    android:name="com.itheima.lifecycle.SecondActivity">
	</activity>

> - 每个应用会有一个Activity任务栈，存放已启动的Activity
> - 即一个应用的activity在启动时按栈的规则存放在栈中。
> - Activity的启动模式，即修改任务栈的排列情况

*  standard 标准启动模式  -> 先进后出
*  singleTop 单一顶部模式 
	如果任务栈的栈顶存在这个要开启的activity，不会重新的创建activity，而是复用已经存在的activity。保证栈顶如果存在，不会重复创建。
*  singeTask 单一任务栈，在当前任务栈里面只能有一个实例存在
	如果该activity没有启动过，会启动跳转到这个activity（即该activity出现在栈顶）
	如果该activity已经启动过，但不知栈顶，即把它前上面的activity全部干掉，它就成为了栈顶activity，与用户交互。
	如果一个activity的创建需要占用大量的系统资源（cpu，内存）一般配置这个activity为singletask的启动模式。w

*  singleInstance启动模式非常特殊， activity会运行在自己的任务栈里面，并且这个任务栈里面只有一个实例存在
	如果你要保证一个activity在整个手机操作系统里面只有一个实例存在，使用singleInstance  例如： 电话拨打界面
->  非单例模式的acticity，如果在10个应用中启动，就会有10个实例（在10个栈中）。
	而单例模式的acticity，在内存中永远只有一个，若10个应用想启动这个activity，就是把该activity所在的任务栈移动至前台。


## 横竖屏切换 ##
>- 默认情况下 ，横竖屏切换， 销毁当前的activity，重新创建一个新的activity。
>- 即横竖屏的切换回触发activity的生命周期方法。  （我们关注这个， 主要是有需求可能在我们进行横竖屏切换时页面显示可能需要改变）

>- 当然一一些情况下我们不希望横竖屏切换activity被销毁重新创建  比如游戏
>- 即我们不想再进行横竖屏切换时调用生命周期相关方法，

1.写死横竖屏：

        <activity
            android:screenOrientation="portrait"    

->or 代码写死（在onCreate()方法中）：

		setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);
2.不去感知横竖屏的变化

        <activity
            android:configChanges="orientation|keyboardHidden|screenSize"
            android:name="com.itheima.lifecycle.MainActivity"
            android:label="@string/app_name" 
            >
	->orientation|keyboardHidden|screenSize 这几种改变都不会触发生命周期相关的方法。

##返回activity时传递数据

###从A界面打开B界面， B界面关闭的时候，返回一个数据给A界面
> 步骤：

- 在开启activity决定我要获取这个activity的返回值

----------

		startActivityForResult(intent, 0);   //0代表请求吗码

-  在新开启的界面里面实现设置数据的逻辑， 即把要返回的数据放到intent中

----------
		Intent data = new Intent();
		data.putExtra("phone", phone);
		//设置一个结果数据，数据会返回给调用者
		setResult(0, data);   //0 位结果码
		finish();//关闭掉当前的activity，才会返回数据

- 在开启者activity里面实现方法
----------
	onActivityResult(int requestCode, int resultCode, Intent data)   
	这个方法会在activity关闭时调用，以获取携带数据的activity的数据。

- 根据请求码和结果码确定业务逻辑

----------

	请求码： 在一个activity中可能会向多个activity索取数据， 这时可以给不同的请求码
	结果码： 同理，一个activity可能会返回不同的结果，

	即在onActivityResult（）中通过这两个码来获取数据，进行合理使用。





 











