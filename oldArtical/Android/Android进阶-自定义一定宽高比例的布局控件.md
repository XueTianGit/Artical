title: Android进阶-自定义一定宽高比例的布局控件
date: 2016/3/3 21:09:19                      
categories: Android
---

# Android进阶-自定义一定宽高比例的布局控件 #
>- 我们可能会有，这种需求：
>    - 我们的控件显示，宽需要适应屏幕(即宽是一定的)， 而我们的高这需要和宽成一定的比例
>    - 那么，这样我们就不用考虑屏幕适配了
>- 下面代码，就自定义了一个这样的布局，可以调整这个布局的宽高比例

>- 代码核心
>    - 重写onMeasure()方法
>        - 重新定义其父容器给他的测量规则
>        - 保证按照自己定义的规则去显示

	//自定的布局， 继承自FrameLayout
	public class RatioLayout extends FrameLayout {
		// 按照宽高比例去显示
		private float ratio = 2.43f; // 比例值，默认
		public void setRatio(float ratio) {
			this.ratio = ratio;
		}	
		public RatioLayout(Context context) {
			super(context);
		}
		//可以使用自定义的属性
		public RatioLayout(Context context, AttributeSet attrs, int defStyle) {
			super(context, attrs, defStyle);
			// 参数1：命名空间   参数2：属性的名字    参数3：默认的值
			float ratio = attrs.getAttributeFloatValue(
					"http://schemas.android.com/apk/res/com.suixin.customview",
					"ratio", 2.43f);
			setRatio(ratio);
		}
	
		public RatioLayout(Context context, AttributeSet attrs) {
			this(context, attrs, 0);
		}
	

		// 测量当前布局
		@Override
		protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
			//获取父控件， 所给测量的规则
			int widthMode = MeasureSpec.getMode(widthMeasureSpec); // 模式
			int widthSize = MeasureSpec.getSize(widthMeasureSpec);// 宽度大小
			int width = widthSize - getPaddingLeft() - getPaddingRight();// 去掉左右两边的padding
	
			int heightMode = MeasureSpec.getMode(heightMeasureSpec); // 模式
			int heightSize = MeasureSpec.getSize(heightMeasureSpec);// 高度大小
			int height = heightSize - getPaddingTop() - getPaddingBottom();// 去掉上下两边的padding
	
			//确定，到底是宽是精确的，还是高是精确的
			if (widthMode == MeasureSpec.EXACTLY
					&& heightMode != MeasureSpec.EXACTLY) {
				// 修正一下 高度的值 让高度=宽度/比例
				height = (int) (width / ratio + 0.5f); // 保证4舍五入
			} else if (widthMode != MeasureSpec.EXACTLY
					&& heightMode == MeasureSpec.EXACTLY) {
				// 由于高度是精确的值 ,宽度随着高度的变化而变化
				width = (int) ((height * ratio) + 0.5f);
			}
			// 重新制作了新的规则
			widthMeasureSpec = MeasureSpec.makeMeasureSpec(MeasureSpec.EXACTLY,
					width + getPaddingLeft() + getPaddingRight());
			heightMeasureSpec = MeasureSpec.makeMeasureSpec(MeasureSpec.EXACTLY,
					height + getPaddingTop() + getPaddingBottom());
	
			super.onMeasure(widthMeasureSpec, heightMeasureSpec);//我就按这个规则来显示
		}
	
	}

>- 自定义的属性

	<resources>
	    <declare-styleable name="com.suixin.customview.view.RatioLayoutt">
	        <attr  name="ratio"  format="float"></attr>
	    </declare-styleable>
	</resources>
		
>- 使用范例：

        <com.suixin.customview.view.RatioLayout
            android:id="@+id/rl_layout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"  //在这里，并不会生效的
            android:padding="5dp" 
            itheima:ratio="2.43"> 

            <ImageView
                android:id="@+id/item_icon"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:scaleType="fitCenter"
                android:src="@drawable/ic_default" />
        </com.suixin.customview.view.RatioLayout>

