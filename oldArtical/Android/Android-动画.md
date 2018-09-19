title: Android-动画
date: 2016/2/29 9:26:25         
categories: Android
---

# Android-动画 #


## 帧动画 AnimationDrawable ##
> - 帧动画在Android2.0时就已经出现了，使用它我可以制作简单的动画效果。
> 因为其简单、轻量。 帧动画现在在Android中还用在很多地方，例如手机的开机动画。

>- 原理：就像传统的动画制作方式一样，一帧一帧画面的快速切换，从而形成动画效果。


使用步骤：

- 在drawable目录下定义xml文件，子节点为animation-list，在这里定义要显示的图片和每张图片的显示时长

----------

		<animation-list xmlns:android="http://schemas.android.com/apk/res/android" android:oneshot="false">
		    <item android:drawable="@drawable/g1" android:duration="200" />
		    <item android:drawable="@drawable/g2" android:duration="200" />
		    <item android:drawable="@drawable/g3" android:duration="200" />
		</animation-list>

- 在屏幕上播放帧动画

----------

		ImageView iv = (ImageView) findViewById(R.id.iv);
		//把帧动画的资源文件指定为iv的背景
		iv.setBackgroundResource(R.drawable.frameanimation);  //frameanimation即为上面定义的xml文件
		//获取iv的背景
		AnimationDrawable ad = (AnimationDrawable) iv.getBackground();
		ad.start();


## Animation（补间动画） ##

> 首先何为补间动画呢？-> 原形态变成新形态时为了过渡变形过程，生成的动画就叫补间动画.
补间动画常用的属性有播放时间、播放次数、重复播放的模式等等。

动画作用在控件上，调用方式通常为： `例如：ImageView.startAnimation(Anamition a)`

- Animation旗下有4个子类：
	`AlphaAnimation,RotateAnimation, ScaleAnimation, TranslateAnimation` 
- 下面来分别看一下：

### TranslateAnimation（位移动画） ###
意思已经很明显了吧。。。。。就是.....

主要来看一下其构造方法：

	1）ta = new TranslateAnimation (float fromXDelta, float toXDelta, float fromYDelta, float toYDelta)。
	参数：
	fromXDelta等4个参数代表起始坐标，结束坐标。参考点为屏幕左上角， 直角坐标系x轴负方向为原x轴正方向，y轴不变（x中坐标的定位，许多都是这个坐标系）。

	2）
	public TranslateAnimation(int fromXType, float fromXValue, int toXType, float toXValue, int fromYType, float fromYValue, int toYType, float toYValue)
	参数：
	fromXType：起始坐标x的值如何解释（就是相对于谁）。Animation.RELATIVE_TO_SELF即相对于自己的多少倍（乘积）
	fromXValue： 起始坐标x的值
	其他参数类似。

范例：

	TranslateAnimation	ta = new TranslateAnimation(Animation.RELATIVE_TO_SELF, -1, Animation.RELATIVE_TO_SELF, 2, 
								Animation.RELATIVE_TO_SELF, -0.5f, Animation.RELATIVE_TO_SELF, 1.5f);

	范例这里设置动画起始x，在x轴负方向一倍处， 移动到x轴正方向2倍处.......


范例：

		ta = new TranslateAnimation(Animation.RELATIVE_TO_SELF, -1, Animation.RELATIVE_TO_SELF, 2, 
				Animation.RELATIVE_TO_SELF, -0.5f, Animation.RELATIVE_TO_SELF, 1.5f);
		//设置播放时间
		ta.setDuration(2000);
		//设置重复次数
		ta.setRepeatCount(1);
		ta.setRepeatMode(Animation.REVERSE);
		iv.startAnimation(ta);    //播放动画

### RotateAnimation（旋转动画） ###
> 位移动画看完其他三种动画就类似了。下面主要都是看一下常用构造方法。

	public RotateAnimation (float fromDegrees, float toDegrees, int pivotXType, 
	                         loat pivotXValue, int pivotYType, float pivotYValue)  
	fromDegrees：起始从多少角度开始
	toDegrees：旋转到多少角度
	pivotXType：说明pivotXValue如何去解释
				可选：Animation.ABSOLUTE、Animation.RELATIVE_TO_SELF、Animation.RELATIVE_TO_PARENT
	pivotXValue：即在哪个x点开始旋转

	范例:
	/*起始角度为0度， 旋转720度， 以自身为参考点（自身左上角为坐标原点）， 在自身中心开始旋转*/
	RotateAnimation ra = new RotateAnimation(0, 720, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);

### ScaleAnimation（缩放动画） ###

	ScaleAnimation sa = new ScaleAnimation(fromX, toX, fromY, toY, iv.getWidth() / 2, iv.getHeight() / 2);
	ScaleAnimation sa = new ScaleAnimation(0.5f, 2, 0.1f, 3, Animation.RELATIVE_TO_SELF, 0.5f,Animation.RELATIVE_TO_SELF, 0.5f);
	这两个应该很好理解了。


### 使补间动画的动画效果一起播放 ###
> 依赖AnimationSet，它也为Animation的小弟。

代码：

		AnimationSet set = new AnimationSet(false);  // false和interpolator这个单词有关
		set.addAnimation(ta);
		set.addAnimation(sa);
		set.addAnimation(ra);
		set.addAnimation(aa);
		iv.startAnimation(set);、

该代码执行时，动画会同时播放

### AlphaAnimation（透明动画） ###
> public AlphaAnimation (float fromAlpha, float toAlpha) 
> 透明度值在0——1之间。 其中0代表全透明， 1代表不透明。

## 属性动画 ObjectAnimator ##
>- 属性动画是Android3.0之后的新特性。
>- 补间动画有一个特点：**他只是改变了页面的绘制效果，而组件的实际位置却没有改变。即组件并没有参与动画**
>- 而属性动画的组件是使组件参与动画效果的。
> - 和补间动画一样属性动画也可以做出位移、旋转等动画效果。


这里截取部分代码：

	平移:
			//x坐标变换4次。（以屏幕为参考）
			ObjectAnimator oa = ObjectAnimator.ofFloat(iv, "translationX", 10, 70, 20, 100);  
	旋转：
			// 0, 180, 90, 360类似平移， 以y轴为中心， 旋转 （以组件自身为参考）
			ObjectAnimator oa = ObjectAnimator.ofFloat(iv, "rotationY", 0, 180, 90, 360);
	缩放：
				
			ObjectAnimator oa = ObjectAnimator.ofFloat(iv, "scaleX", 1, 1.6f, 1.2f, 2);
	透明：
			ObjectAnimator oa = ObjectAnimator.ofFloat(iv, "alpha", 0, 0.6f, 0.2f, 1);
	
	属性动画的播放：
				//oa.setDuration(..);
				//oa.setRepeatCount(..);
				//oa.setRepeatMode(..);
				oa.start();
















  

