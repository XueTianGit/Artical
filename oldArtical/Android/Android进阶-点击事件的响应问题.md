title: Android进阶-点击事件的响应问题
date: 2016/3/3 21:10:41                          
categories: Android
---

# Android进阶-点击事件的响应问题 #

> 为什么我们有时在XML文件中明明给一个控件设置了点击事件却不响应？
> 如何制止ListView的点击事件被抢走？


- 不同的控件Android系统对于其点击事件的默认时不同的
	- 有些控件默认是不可以点击的例如TextView
	- 如果想要在XML中配置这种控件响应点击事件应配置两个属性
		- clickable = "true"
		- onClick = "eventName"
	- 对于代码中直接通过View.setOnClickListener()，那么即使想TextView这样的控件也可以响应，这是因为：

	Android源码是这样处理的：
	
	    public void setOnClickListener(OnClickListener l){
	        if(!isClickable()){
	            setClickable(true);
	        }
	        mOnClickListener = l;
   		 }

		- 即只要View.setOnClickListener()这样设置了的话，会强行置为true

- 当控件的点击事件覆盖ListView的点击事件是，应把控件的这两属性设置一下
	- Clickable = "false",  focusable = "false"


- 对于ListView这种控件的使用
	- 我们应该把ListView要展现的界面内容完全封装，这样，当我们想要改变界面数据显示是，只需改变
	- 数据即可，而不需要去修改UI代码


- CheckBox的选择器
	- 正常情况下，给一个控件设置选择器是使用 background = "@drawable/selector"
	- 但是CheckBox的选择器有些特殊他是  button = "@drawable/selector"