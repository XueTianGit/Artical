title: Android进阶-Android自带API-Demo与震动器
date: 2016/3/1 8:42:11   
categories: Android
---

# Android进阶-Android自带API-Demo与震动器 #

## API-Demo ##

> - 在android-sdk\samples\android-14\ApiDemos下有许多Android为他的特性提供的Demo。
> 在学习android时， 我们可以经常去看看这个API Demo，  看看有什么我们感兴趣， 可以学习的东西， 然后单独学习一下。
> 可以把APIDemo 导入到eclipse中， 然后安装到真机上，以便学习。
> 
> - 如果我们想查询APIDemo中相关代码， 可以使用eclipse中的 search file功能

### Shnake动画与插补器 ###

API-Demo给出了Shnake动画的演示， 例如让EditText来回摆动。
按照Demo中给出的范例，我们可以这样使用：

在res/ 目录下创建shake.xml和cycle_7.xml     （shake.xml用来定义shake动画，cycle_7.xml中定义了相关的插补器）
		
		<!--shake.xml-->
		<translate xmlns:android="http://schemas.android.com/apk/res/android"
		    android:duration="1000"
		    android:fromXDelta="0"
		    android:interpolator="@anim/cycle_7"
		    android:toXDelta="10" />
		<!--cycle_7.xml-->
		<cycleInterpolator xmlns:android="http://schemas.android.com/apk/res/android" android:cycles="7" />

	-代码中使用shnake动画
		Animation shake = AnimationUtils.loadAnimation(this, R.anim.shake);
		EditText.startAnimation(shake);


- 通过上面这个范例， 可以来看以下插补器是干什么的。
- 拿shake动画为例， shake动画是左右摇晃， 那么这个摇晃也是有说法的， 你的摇晃的轨迹与速度是个什么关系呢？
- 即插补器可以理解为，定义运动轨迹与速度的关系   -> 在高中数学看来， 就可以理解为斜率（加速度为多少）    >  可以看出数学好的程序员， 又会到另一个更高的层次了， 小伙， 好好学数学！！！


> - Android中已经定义好了一些插补器， 例如：
> - LinearInterpolator 直线插补器（匀速）
> - DecelerateInterpolator 减速插补器（先快后慢）
> - AccelerateInterpolator 加速插补器（先慢后快）
> - AccelerateDecelerateInterpolator （效果）加速减速插补器（先慢后快再慢）


上面这些插补器的数学方程你会写吗？？？？？

- 简单自定义插补器


----------
	
	 shake.setInterpolator(new Interpolator() {
		
		 @Override   //y = x
		 public float getInterpolation(float x) {

			 return x;
		 }
	 });


----------

- 文本框监听器
	当我们想监听文本框内容，并作出实时反应的话，可以使用这个

	EditText.addTextChangedListener(new TextWatcher() {
			
			@Override
			public void onTextChanged(CharSequence s, int start, int before, int count) {
				String queryNumber = s.toString();
					/*do something*/
			}
	
- 震动器 Vibrator

使用它我们可以去使手机震动。使用它需要获得权限： android.permission.VIBRATE
范例：
		Vibrator vibrator = (Vibrator) getSystemService(VIBRATOR_SERVICE);  
		// vibrator.vibrate(2000);震动两秒

		//参2表示从第几个位置开始循环
		vibrator.vibrate(new long[] { 1000, 2000, 1000, 3000 }, -1);    // 先等待1秒,再震动2秒,再等待1秒,再震动3秒,
																		// 参2等于-1表示只执行一次,不循环,
																		// 参2等于0表示从头循环,
		
		// 取消震动vibrator.cancel()

