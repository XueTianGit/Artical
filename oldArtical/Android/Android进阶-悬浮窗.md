title: Android进阶-悬浮窗
date: 2016/3/1 19:14:38      
categories: Android
---
# Android进阶-悬浮窗 #

需求如标题，那么怎么做出这个效果呢？ -> 主要依赖WindowManeger

> - 我们使用WindowManeger,也可以把自己定义的一个控件（悬浮窗），可以在其他应用最上层，甚至手机桌面最上层显示窗口。
> - 调用的是WindowManager继承自基类的addView方法和removeView方法来显示和隐藏窗口
> - 悬浮窗口并不受activity的影响，他是隶属于启动它的应用程序所在进程。当进程挂掉，悬浮窗也会挂掉

-  每一个WindowManager对象都和一个特定的 Display绑定。
- 想要获取一个不同的display的WindowManager，可以用 createDisplayContext(Display)来获取那个display的 Context，、
- 之后再使用： Context.getSystemService(Context.WINDOW_SERVICE)来获取WindowManager。

> 下面代码自定义了一个Toast代码要点：

- 使用WindowManager必须先获得权限： android.permission.SYSTEM_ALERT_WINDOW
- 代码中获取的getDefaultDisplay()， 可以理解为屏幕
- params是用来设置我们添加的控件的布局， 可以理解为xml文件中的布局属性
- 控件的移除可使用removeView方法
- params.gravity = Gravity.LEFT + Gravity.TOP;// 将重心位置设置为左上方,这样控件在布局时，会以屏幕的左上角为坐标原点

----------

	/**
	 * 自定义归属地浮窗 需要权限android.permission.SYSTEM_ALERT_WINDOW
	 */
	private void showToast(String text) {
		mWM = (WindowManager) this.getSystemService(Context.WINDOW_SERVICE);

		// 获取屏幕宽高
		winWidth = mWM.getDefaultDisplay().getWidth();
		winHeight = mWM.getDefaultDisplay().getHeight();

		params = new WindowManager.LayoutParams();
		params.height = WindowManager.LayoutParams.WRAP_CONTENT;
		params.width = WindowManager.LayoutParams.WRAP_CONTENT;
		params.flags = WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
				| WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON;
		params.format = PixelFormat.TRANSLUCENT;
		params.type = WindowManager.LayoutParams.TYPE_PHONE;// 电话窗口。它用于电话交互（特别是呼入）。它置于所有应用程序之上，状态栏之下。
		params.gravity = Gravity.LEFT + Gravity.TOP;// 将重心位置设置为左上方,
													// 也就是(0,0)从左上方开始,而不是默认的重心位置
		params.setTitle("Toast");


		TextView tvText = (TextView) view.findViewById(R.id.tv_number);
		tvText.setText(text);

		mWM.addView(view, params);// 将view添加在屏幕上(Window)
	
	}

