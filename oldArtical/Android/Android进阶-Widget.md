title: Android项目-Widget
date: 2016/3/7 18:25:50   
categories: Android
---

# Android项目-Widget #

> App Widgets是一个小的应用控件，它能够嵌入在其他的应用中（像主屏幕），并且可以周期性的更新。
> 能够拥有widget的应用程序被叫做App Widget host，
	

- 简单创建widget  (具体内容在开发文档中写的很详细)
	- 1.创建一个AppWidget类，继承AppWidgetProvider（这个类继承自BroadcastReceiver）
	- 2.在manifest文件中声明上面创建的AppWidget, 即声明<receiver/>
	- 3.在xml目录下，定义widget的元数据
	- 4.定义widget的布局文件

## Widget的细节 ##

- 在配置widget的元数据时， 
	- 如果widget的最小宽度大于屏幕的宽度，那么这个widget是不会被显示的
	- widget的默认更新时间是没个半个小时，如果我们设置android:updatePeriodMillis="0"， 那么我们就可以自己控制这个更新时间。
- widget的生命周期
	- onUpdate ： Widget在更新时调用的方法
	- onDelete ： 每当一个widget从桌面删除时会调用
	- onReceive： 当接收到Widget操作时首先调用的是OnReceive方法，然后才是相关的操作方法。可以说由这个方法去调用widget的其他的生命周期方法
	- onEnabled：此方法在Widget第一次被创建的时候调用，并且只调用一次，此方法中常放入初始化数据，服务的操作。
	- onDisabled：所用Widget被删除是调用的方法，同onEnabled方法相对

- widget的生命周期方法中不能做耗时操作
	- 这是因为AppWidgetProvider实际上继承自BroadcastReceiver，要知道的是广播的onReceive()方法必须自10秒内完成，因此widget的生命周期方法也得在10内完成
	- 一般，我们可以开启一个服务去做业务逻辑相关事情

## 操作widget ##
> 既然把widget放在桌面上，我们肯定是要使用它来展示数据的，但是桌面是别人的App（例如 android launch2），即我们的widge是展示在别人的应用中的，
> 那么我们如何操作它呢？

要完成上面的工作，主要与这3个对象有关：

- AppWidgetManager
	- 可以利用其updateAppWidget(ComponentName, RemoteViews)方法；来更新我们的widget



以下面代码为例：

这是位于以个服务的onCreate()方法中
	@Override
	public void onCreate() {
		super.onCreate();
		
		//桌面小控件的管理者	
		widgetManager = AppWidgetManager.getInstance(this);
		
		//下面要使用一个定时器，来周期性更新widget
		timer = new Timer();
		timerTask = new TimerTask() {		
			@Override
			public void run() {
				


				/**
				 * 由于更新的需要，这里初始化一个RemoteViews
				 * 包名即我们的应用程序的包名， 布局文件即我们的widget的布局文件
				 */
				RemoteViews views = new RemoteViews(getPackageName(), R.layout.process_widget);
				/**
				 * 需要注意的是，RemoteViews并没有findViewById()方法。
				 * 这里需要使用下面类似的方法来更新我们的widget界面控件显示
				 */
				views.setTextViewText(R.id.process_count,"正在运行的软件: 10"); 
				views.setTextViewText(R.id.process_memory, "可用内存:1G" );


				
				/*
				 *当我们的widget，响应点击事件时，也是不可以像以前那样处理， 可以通过启动一个广播、服务或者Activity的方式来响应我们的点击事件
				 *这里，启动一个广播
				 */
				Intent intent = new Intent();	
				//发送一个隐式意图
				intent.setAction("com.suixin.mobileguard.response_widget_receicer");	
				PendingIntent pendingIntent = PendingIntent.getBroadcast(getApplicationContext(), 0, intent, 0);
				//设置点击事件
				views.setOnClickPendingIntent(R.id.btn_clear, pendingIntent);

				
				//第一个参数表示上下文
				//第二个参数表示我们的widget对应类（继承AppWidgetProvider）
				ComponentName provider = new ComponentName(getApplicationContext(), MyAppWidget.class);
				//更新桌面
				widgetManager.updateAppWidget(provider, views);			
			}
		};

		//从0开始。每隔5秒钟更新一次
		timer.schedule(timerTask, 0, 5000)；	
	}


	
