title: Android进阶-子线程中刷新UI的讨论
date: 2016/3/1 19:14:57       
categories: Android
---
# Android进阶-子线程中刷新UI的讨论 #

我们经常会遇到这个错误 -> 不可以在主线程之外更新UI 
> android.view.ViewRoot$CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views。

一般，我们将耗时操作，都放在子线程中。但是：

## 子线程中能不能更新UI呢？ ##
- 首先，Google是不推荐在子线程中更新UI的，对于主线程（UI），
	- Do not block the UI thread
    - Do not access the Android UI toolkit from outside the UI thread 
  	- 但是其实在子线程中也是可以更新UI， 那是在什么情况下可以呢？
	  	- 当在Activity的onCreate()方法刚开始时，并且子线程中逻辑简单时，就可以更新UI	
	  	- 这是因为，Android系统内部有一个类会去检查UI刷新的线程。（这个类的 invalidate方法回去检查是否可以更新UI）
	  	- 那么，在这个类还没有初识化成功时，我们是可以更新UI的。
	- 即我们是可以在子线程更新UI的，但条件是非常苛刻的，靠运气

## 如何在子线程中更新UI ##

- 使用Handler + message
	- 场合：如果是后台任务，像是下载任务等，就需要使用AsyncTask。 如果需要传递状态值等信息，像是蓝牙编程中的socket连接，就需要利用状态值来提示连接状态以及做相应的处理，
	- 就可以使用这种方式
- 使用


----------
	
	Activity.runOnUiThread(Runnable runnable)
	new Thread() {
        public void run() {
            //这儿是耗时操作，完成之后更新UI；
            runOnUiThread(new Runnable(){

                @Override
                public void run() {
                    //更新UI
                    imageView.setImageBitmap(bitmap);
                }
            });
        }
    }.start()

- View.post(Runnable r)
	- Causes the Runnable to be added to the message queue, to be run after the specified amount of time elapses. 
	- The runnable will be run on the user interface thread
	- 即这样，Runnable会运行在UI线程， 不过，时间可能延迟，



## 如何延迟线程 ##

1. Handler.postDelayed()

----------

	public final boolean postDelayed (Runnable r, long delayMillis)
	这个方法我们可以利用他来做延迟，但是不能用来在子线程中更新UI。
	例如，在主线程中延迟
	        new Handler().postDelayed(new Runnable() {
	            @Override
	                         public void run() {
	
	            }
	        }, 2000);   //用来做延迟， 还酬和
	
	延迟两秒执行Runnable, 这个Runnable是在创建Handler的线程中执行的。

2. Thread.sleep(2000);
	- 这个方法很普通，就是进行线程延时， 但是要处理异常（代码变丑了）

3. SystemClock.sleep(2000);
	- 这个方法实际上是对Thread.sleep();的封装，只不过帮我们把异常处理给解决了