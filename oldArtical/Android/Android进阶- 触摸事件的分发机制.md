title: Android进阶- 触摸事件的分发机制
date: 2016/3/1 9:18:40               
categories: Android
---

# Android进阶- 触摸事件的分发机制#

- 先来看一下，触摸事件传递的3个方法
	- onInteceptTouchEvent(): 返回true表示拦截这次触摸事件， false表示不拦截
	- dispatchTouchEvent()： 用来分发事件， 如果事件被拦截则交给
	- onTouchEvent()： 处理触摸事件，返回true表示事件被消耗， false表示没有对触摸事件进行处理   

- 那么，Android的触摸事件是怎样分发的呢

![](http://7xrbxa.com1.z0.glb.clouddn.com/android%E8%A7%A6%E6%91%B8%E4%BA%8B%E4%BB%B6%E7%9A%84%E5%88%86%E5%8F%91%E6%9C%BA%E5%88%B6.png)

![](http://7xrbxa.com1.z0.glb.clouddn.com/android%E8%A7%A6%E6%91%B8%E4%BA%8B%E4%BB%B6%E7%9A%84%E5%88%86%E5%8F%91%E6%9C%BA%E5%88%B62.png)

> 可以看出，事件会优先交给父控件来处理， 但是这样的话，做儿子的是不是也太没有权利了，因此，为了给儿子一点机会， 
> 在dispathchTouchEvent()中可以这样写：
		getParent().requestDisallowInterceptTouchEvent(true);  //请求父控件给它处理触摸事件的机会

**在anroid中ViewGroup一般是将事件优先交给其儿子来处理的**