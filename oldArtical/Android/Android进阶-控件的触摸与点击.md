title: Android进阶-控件的触摸与点击
date: 2016/3/7 18:26:50     
categories: Android
---

# Android进阶-控件的触摸与点击 #

- Point1

> 在Android中，onClick、onLongClick的触发是和ACTION_DOWN及ACTION_UP相关的，在时序上，如果我们在一个View中同时覆写了onClick、
> onLongClick及onTouchEvent的话，onTouchEvent是最先捕捉到ACTION_DOWN和ACTION_UP事件的，其次才可能触发onClick或者onLongClick。

在Android的源码中是这样写的：
即事件顺序为: ACTION_DOWN -> onLongClick -> ACTION_UP -> onClick

	case MotionEvent.ACTION_DOWN:
	
	    mPrivateFlags |= PRESSED;
	
	    refreshDrawableState();
	
	    if ((mViewFlags & LONG_CLICKABLE) == LONG_CLICKABLE) {
	
	         postCheckForLongClick(); 
	
	    }
	
	    break;
	
	case MotionEvent.ACTION_UP:
	
	    if ((mPrivateFlags & PRESSED) != 0) {
	
	         boolean focusTaken = false;
	
	         if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
	
	               focusTaken = requestFocus();
	
	         }
	
	
	    if (!mHasPerformedLongPress) {
	
	       if (mPendingCheckForLongPress != null) {
	
	             removeCallbacks(mPendingCheckForLongPress);
	
	       }
	
	       if (!focusTaken) {
	
	              performClick();
	
	       }
	
	    }
	
	    …
	
	    break;

- 即在开发时应注意
	- 一个控件响应触摸或者点击事件，如果返回true, 则这个事件，就算已经处理完了， 如果返回false，那么事件会按照顺序继续传递。
	- 因此，触摸与点击同时监听时， 如果触摸返回了true， 那么点击监听是不可能再监听到这个事件发生的。


- Point2
	- 如果想要用户，可以看到界面，但是却不能操作界面，（类似第一次安装软件的引导使用）。
	- 我们可以搞一个透明类型的activity覆盖在上面，这样就可以达到效果。

例如：
	
	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent" 
	    android:background="#5000"    <!--半透明-->
	    > </RelativeLayout>

- Point3
	- 当我们在service中启动Activity时，应设置FLAG_ACTIVITY_NEW_TASK
	- 这是因为Service并没有任务栈， 而Activity想要运行，是需要任务栈的，因此要为Activity弄一个任务栈
