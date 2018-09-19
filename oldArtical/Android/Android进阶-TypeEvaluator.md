title: Android进阶-TypeEvaluator
date: 2016/3/3 21:08:28                      
categories: Android
---

# Android进阶-TypeEvaluator #
>- 类型估值器
>- 可以说他的功能是： 根据当前百分比，计算从开始值到结束值中间的值。
>- 类型估值器一般在我们处理动画时可能会用到
>-  Android自带的估值器有： ArgbEvaluator, FloatArrayEvaluator, FloatEvaluator, IntArrayEvaluator, IntEvaluator, PointFEvaluator, RectEvaluator 
>- 其实其核心就是下面这一段代码：

	//fraction， 当前百分比
	public Float evaluate(float fraction, Number startValue, Number endValue) {
		float startFloat = startValue.floatValue();
		return startFloat + fraction * (endValue.floatValue() - startFloat);  //根据百分比，得到当前的进度值
	}

>- 由这个可以扩展到很多估值
>- 比如颜色变化的过渡值
>- 控件移动过程中，位置的变化值等等

	/**
	 * 颜色变化过度
	 */
	public Object evaluateColor(float fraction, Object startValue,
			Object endValue) {
		int startInt = (Integer) startValue;
		int startA = (startInt >> 24) & 0xff;
		int startR = (startInt >> 16) & 0xff;
		int startG = (startInt >> 8) & 0xff;
		int startB = startInt & 0xff;

		int endInt = (Integer) endValue;
		int endA = (endInt >> 24) & 0xff;
		int endR = (endInt >> 16) & 0xff;
		int endG = (endInt >> 8) & 0xff;
		int endB = endInt & 0xff;

		return (int) ((startA + (int) (fraction * (endA - startA))) << 24)
				| (int) ((startR + (int) (fraction * (endR - startR))) << 16)
				| (int) ((startG + (int) (fraction * (endG - startG))) << 8)
				| (int) ((startB + (int) (fraction * (endB - startB))));
	}
