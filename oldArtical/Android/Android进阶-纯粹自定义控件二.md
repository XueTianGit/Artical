title: Android进阶-纯粹自定义控件二
date: 2016/3/3 21:07:05                   
categories: Android
---

# Android进阶-纯粹自定义控件二#

本文来看一下自定义ViewGroup需要注意哪些。
以自定义的一个侧滑菜单为例。
图例：

![](http://7xrbxa.com1.z0.glb.clouddn.com/android%E7%BA%AF%E7%B2%B9%E8%87%AA%E5%AE%9A%E4%B9%89%E6%8E%A7%E4%BB%B6%E4%B8%801.png)
![](http://7xrbxa.com1.z0.glb.clouddn.com/android%E7%BA%AF%E7%B2%B9%E8%87%AA%E5%AE%9A%E4%B9%89%E6%8E%A7%E4%BB%B6%E4%B8%802.png)
![](http://7xrbxa.com1.z0.glb.clouddn.com/android%E7%BA%AF%E7%B2%B9%E8%87%AA%E5%AE%9A%E4%B9%89%E6%8E%A7%E4%BB%B6%E4%B8%803.pngg)
![](http://7xrbxa.com1.z0.glb.clouddn.com/android%E7%BA%AF%E7%B2%B9%E8%87%AA%E5%AE%9A%E4%B9%89%E6%8E%A7%E4%BB%B6%E4%B8%804.png)


- 关键点
>- 既然是自定义的ViewGroup， 那么的话，控件的具体内容肯定不是要考虑的事情
>- 这个ViewGroup应考虑的是
>     - 我们这个ViewGroup有何特点？ -> 子View的行为
>     - 如何完成自己的 onMeasure(), onLayout()方法，从而控制子View
>     - 如何去实现子View的行为

## 侧滑菜单 ##

- 这个ViewGroup所具有的特点为：
	- 拥有两个子View，正常状态下，一个可见，一个不可见
	- 可以通过滑动，来控制子View的显示
	- 可以给外界暴露一个借口，来控制隐藏的View显示 


### 实现关键点 ###

- 如何得到得到子View的引用，从而实现对子View的控制
	- 首先，在构造方法中是得不到的，构造知识把自己给弄出来了
	- 可以覆写protected void onFinishInflate()
		- 这个方法会在所有的子View被inflate之后调用
		- 因此可以通过ViewGroup.getChildAt(position)来得到子View的引用
	
- onMeasure方法
	- protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)
	- widthMeasureSpec和heightMeasureSpec在前面已经说过，这是View的父View传递给他的测量值
	- 我们可以利用这两个参数，来对子View执行测量
	- 不过，在自定义ViewGroup方法时，一般重写这个方法，因为这个方法主要就是用来测量出（依据父View和布局文件中的参数）子View的宽高
	- 我们可以借用其他的ViewGroup的方法， 因此我们可以通过继承来实现（例如继承FrameLayout）
	- 来看一下系统是如何对View进行测量的（我们也可以这么干）
		- 系统测量依赖 MeasureSpec 的 public static int makeMeasureSpec (int size, int mode)方法
		- size代表在布局文件中的宽高的值， mode代表解析这个宽高的模式
		- mode可取： UNSPECIFIED(wrap_content)   EXACTLY（精确规定了大小，例如多少dp）  AT_MOST（比如math_parent）
		- 例如： int measureSpecWidth = MeasureSpec.makeMeasureSpec(childViewWidth, MeasureSpec.EXACTLY);

- 如何使一个View隐藏，一个View展示
	- 这肯定是关于子View如何布局，即我们自定义的这个ViewGroup在最初对子View进行布局时， 一个充满屏幕（父View），一个在屏幕外面
	- 上面的实现应在onLayout()方法中
	
可以这样实现：

	@Override
	protected void onLayout(boolean changed, int l, int t, int r, int b) {
		menuView.layout(-menuWidth, 0, 0, menuView.getMeasuredHeight());  //在外面
		mainView.layout(0, 0, r, b); //充满父View
	}


- 处理用户滑动，两个View显示的问题
	- 当用户向右滑动是，肯定是要是menuView显示出来
		- 有3中方法可以做到这一点（控制子View的移动显示）
			- layout(l, t, r, b)  很好理解，再布一次局就是了
			- offsetTopAndBottom()和offsetLeftAndRight()  
			- scrollTo() 
				- 将View的边框向(x, y)滑动， 会引起重绘， 这个x, y并没什么限制， 就向在一张大纸上滑动
				- getScrollX ()， 得到已经滑动的距离

	- 使用scrollTo()方法来完成子View的滑动效果
		- 我们滑动的是ViewGroup的边框（屏幕的边框）
		- 由相对论，我们可以得出，我们向右滑动，右边的内容就显示出来了，想左滑动，左边的内容就显示出来了
		- 反正就是，你想把左边的隐藏的menuView给滑动出来， 调用scrollTo()时就得向左滑！（得给负值）

	- 即我们在触摸滑动中可以这样处理
		如下

			//向右滑slideLength 是正的， 向左滑slideLength是负的
			int newScrollX = getScrollX() - slideLength；	  //上次滑动的距离 - 这次滑动的距离
			if(newScrollX<-menuWidth)
                newScrollX = -menuWidth;     //最大滑动距离， 不能超出menuView的宽
			if(newScrollX>0)newScrollX = 0;	 //		不能向左滑
			scrollTo(newScrollX, 0);      

- 暴露出给用户直接显示menuView和关闭menuView的接口
	- 其实这两个接口，就是能够使用户通过调用这两个接口来完成menuView的显示
	- 直接实现的话很简单， 就向上面的代码， 滑出来就是
	- 但是， 一次滑动到指定位置，太快，用户体验不好
	- 因此我们需要，让menuView在一定的时间内滑动出来哦
	- 实现这个问题，共有三种办法


1）使用自定义动画， 
	
	/**
	 * 这个动画让指定view在一段时间内scrollTo到指定位置
	 */
	public class ScrollAnimation extends Animation{
		
		private View view;
		private int targetScrollX;
		private int startScrollX;
		private int totalValue;
		
		public ScrollAnimation(View view, int targetScrollX) {
			super();
			this.view = view;
			this.targetScrollX = targetScrollX;
			
			startScrollX = view.getScrollX();
			totalValue = this.targetScrollX - startScrollX;
			
			int time = Math.abs(totalValue);  
			setDuration(time);      //实现，依据滑动的距离，来决定滑动动画时间的长与短
		}
	
	
	
		/**
		 * 在指定的时间内一直执行该方法，直到动画结束
		 * interpolatedTime：0-1  标识动画执行的进度或者百分比
		 * 当前的值 = 起始值 + 总的差值*interpolatedTime
		 */
		@Override
		protected void applyTransformation(float interpolatedTime,
				Transformation t) {
			super.applyTransformation(interpolatedTime, t);
			int currentScrollX = (int) (startScrollX + totalValue*interpolatedTime);
			view.scrollTo(currentScrollX, 0);
		}
	}

2）利用Scroller， 它可以模拟一个移动的过程，但并不会是View真正移动

	private void closeMenu(){
		scroller.startScroll(getScrollX(), 0, 0-getScrollX(), 0, 400);
		invalidate();
	}
	
	private void openMenu(){
		scroller.startScroll(getScrollX(), 0, -menuWidth-getScrollX(), 0, 400);
		invalidate();
	}
	
	/**
	 * Scroller不主动去调用这个方法
	 * 而invalidate()可以掉这个方法
	 * invalidate->draw->computeScroll
	 */
	@Override
	public void computeScroll() {  //这个方法
		super.computeScroll();
		if(scroller.computeScrollOffset()){//返回true,表示动画没结束， 当计算的滑动的值到我们模拟滑动的地方就停止
			scrollTo(scroller.getCurrX(), 0);
			invalidate();   
		}
	}

3）使用值动画
>- 值动画，可以模拟一个数学过程， 使用方法如下代码所示


	ValueAnimator animator = ValueAnimator.ofInt(0,100);  
	animator.addUpdateListener(new AnimatorUpdateListener() {  // 监听值的变化
					
					@Override
					public void onAnimationUpdate(ValueAnimator animator) {  //在这里做一些事情
						/*....*/
					}
				});
	animator.setDuration(500);
	animator.start();
	 






