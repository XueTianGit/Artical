title: Android进阶-ViewDragHelper
date: 2016/3/3 21:08:05                     
categories: Android
---

# Android进阶-ViewDragHelper #

>ViewDragHelper是Google2013 IO大会提出的，用它来解决界面控件拖拽和移动的问题非常简单。
>因此，可以使用它来自定义ViewGroup，事件处理变的非常简单
>下面来看以下怎么使用

## 一般使用的方法 ##
- 对象的初始化
>- ViewDragHelper位于support-v4包下，可以利用下面的静态方法获得。
	
	//forParent 代表ViewDragHelper驱动的父View
	//sensitivity 表示对控件拖拽响应的敏感度，范围为 1-1.0f， 默认为1.0f
	//cb   ViewDragHelper处理控件拖拽事件的回调函数
	public static ViewDragHelper create(ViewGroup forParent, Callback cb)；
	public static ViewDragHelper create(ViewGroup forParent, float sensitivity, Callback cb)


- 将控件的触摸事件交给ViewDragHelper来处理
>
	@Override
	public boolean onInterceptTouchEvent(MotionEvent ev) {
		// 传递给mDragHelper
		return mDragHelper.shouldInterceptTouchEvent(ev);
	}
	@Override
	public boolean onTouchEvent(MotionEvent event) {
		try {
			mDragHelper.processTouchEvent(event);
		} catch (Exception e) {
			e.printStackTrace();
		}
		// 返回true, 持续接受事件
		return true;
	}

- 在ViewDragHelper的回调函数中处理触摸拖拽事件
>- 比较重要的，常用的回调函数如下
>- 下面方法是按其调用顺序排列的

	ViewDragHelper.Callback mCallback = new ViewDragHelper.Callback() {
		
		// 1. 根据返回结果决定当前child是否可以拖拽
		// child 当前被拖拽的View
		// pointerId 区分多点触摸的id
		@Override
		public boolean tryCaptureView(View child, int pointerId) {
			return true;  //当前的ViewGroup的所有孩子都可以拖拽
		};
		

		//2. 当capturedChild被捕获时,调用.
		@Override
		public void onViewCaptured(View capturedChild, int activePointerId) {
			Log.d(TAG, "onViewCaptured: " + capturedChild);
			super.onViewCaptured(capturedChild, activePointerId);
		}

		//3. 返回控件拖拽的范围, 不对拖拽进行真正的限制. 仅仅决定了动画执行速度
		@Override
		public int getViewHorizontalDragRange(View child) {
			return mRange;
		}
		
		//4. 根据建议值 修正将要移动到的(横向)位置  
		// 此时没有发生真正的移动！！！
		// child: 当前拖拽的View
		// left 新的位置的建议值（即控件要移动到什么地方）, dx 位置变化量
		// left = oldLeft + dx;
		public int clampViewPositionHorizontal(View child, int left, int dx) {
			return left;  //直接返回left表示，就按建议值确定控件将要移动的位置（即，接下来就要按这个位置移动控件了）
		}

		// 5. 当View位置改变的时候, 处理要做的事情 
		// 此时,View已经发生了位置的改变！！！！
		//这个方法调用频率较高
		// changedView 改变位置的View
		// left 新的左边值(控件已经来到了这个left)
		// dx 水平方向变化量
		@Override
		public void onViewPositionChanged(View changedView, int left, int top,
				int dx, int dy) {			
			//一般在这个方法中：(更新状态, 伴随动画, 重绘界面)
		}



		// 6. 当View被释放的时候调用。
		// float xvel View被释放时水平方向的速度, 向右为+
		// float yvel View被释放时竖直方向的速度, 向下为+
		@Override
		public void onViewReleased(View releasedChild, float xvel, float yvel) {		
			//可以在这里， 处理的事情(执行动画)
		}

		@Override
		public void onViewDragStateChanged(int state) {
			// TODO Auto-generated method stub
			super.onViewDragStateChanged(state);
		}

	};

## 提供的其他功能 ##

- 利用ViewDragHelper来处理控件的平滑移动
>这是一个模板代码，记住怎么写就行的，如下


	//public boolean smoothSlideViewTo(View child, int finalLeft, int finalTop)
	//mMainContent： 即平滑动画作用的控件
	//finalLeft ：  控件最终要滑动到的位置 left
	//0（finalTop）： 控件最终要滑动到的位置的top
	//1. 首先，利用这个代码，来触发一个控件的平滑动画
	if(mDragHelper.smoothSlideViewTo(mMainContent, finalLeft, 0)){
				// 返回true代表还没有移动到指定位置, 需要刷新界面.
				// 参数传this(child所在的ViewGroup)
				ViewCompat.postInvalidateOnAnimation(this);  //重绘控件的位置，即滑动， 会引起computeScroll()的调用
	}

     * If this method returns true, the caller should invoke {@link #continueSettling(boolean)}
     * on each subsequent frame to continue the motion until it returns false. If this method
     * returns false there is no further work to do to complete the movement.

	//2.重写View的computeScroll()方法，来使滑动继续	
	@Override
	public void computeScroll() {
		super.computeScroll();	
		// 2. 持续平滑动画 (高频率调用)
		if(mDragHelper.continueSettling(true)){ //  如果返回true, 动画还需要继续执行
			ViewCompat.postInvalidateOnAnimation(this);
		}
	}