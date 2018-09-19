title: Android-Fragment
date: 2016/2/29 8:09:49    
categories: Android
---

# Android-Fragment #

写的很好的Fragment分析文章：

[http://blog.csdn.net/lmj623565791/article/details/37970961](http://blog.csdn.net/lmj623565791/article/details/37970961)

[http://blog.csdn.net/lmj623565791/article/details/37992017](http://blog.csdn.net/lmj623565791/article/details/37992017)


- Fragment是干什么用的？

----------

	感觉可以把Fragment理解为一个小型的Activity， 不过他依赖于Activity。（	每一个fragments 都有自己的一套生命周期回调方法和处理自己的用户输入事件）
	以前我们会把页面布局直接写在Activity的布局文件中， 往往内容不少，而Fragment的出现，
	我们可以把一部分页面布局写在Fragment中， Activity就成了一个总控制器，它只需在布局文件中
	对fragment进行布局即可， 想到这里， fragment是不是很类似于网页布局中的framge呢？
	Fragment的出现，不仅分散了activity布局，还可以使activity的布局更加丰富多彩吧？

- 既然是activity的小弟，Fragment的生命周期方法类似于activity

----------

	即fragment的生命周期，依赖于Activity,官方文档是这样描述的：
	he fragment's lifecycle is directly affected by the host activity's lifecycle. For example, 
	when the activity is paused, so are all fragments in it, and when the activity is destroyed, so are all fragments. 
	但是，Activity处于可见状态时， 可以任意操作Fragment，例如把它移除

- Fragment的简单使用步骤

----------

	a, 创建Fragement对象
	b，获取Fragment管理器
	c，打开Fragment事务
	d，把Fragment显示到Activity
	e，提交事务

范例：

		//Activity中：
	  protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	     
	        fg3 = new Fragment03();
	    	//获取fragment管理器
	    	FragmentManager fm = getFragmentManager();
	    	//打开事务
	    	FragmentTransaction ft = fm.beginTransaction();
	    	//把内容显示至帧布局
	    	ft.replace(R.id.fl, fg3);
	    	//提交
	    	ft.commit();
	    }
	
		//Fragment03
		public View onCreateView(LayoutInflater inflater, ViewGroup container,
				Bundle savedInstanceState) {
			// TODO Auto-generated method stub
			View v = inflater.inflate(R.layout.fragment03, null);  //将Fragment布局文件，转换成view对象，以便索引fragment布局文件中的组件
			return v;
		}
		
- Fragment与其大哥Activity之间是可以相互传递数据的。
> 	代码体现当然是对象间的相互访问呗。。

- FindViewById（）方法在调用时就有特殊性的了，
> - 	在Fragement中调用， 就在Fragment中找；
> - 	在Activity中调用， 就在Activity中找

- 在使用Fragment时要注意器向下的兼容性， 因为Fragment是Android3.0之后的新特性，

----------

	所以较低Android版本是没有Fragment的相关jar的。
	但是可以引入v4的包，然后Activity继承FragmentActivity，然后通过getSupportFragmentManager获得FragmentManager。
	不过还是建议版Menifest文件的uses-sdk的minSdkVersion和targetSdkVersion都改为11以上，这样就不必引入v4包了

- 在Activity的布局文件中，我们可以把Fragment填充在任何一个属于ViewGroup的布局中

----------

	为把Fragment显示在activity中，使用：   FragmentTransaction.replace(ViewGroupId, FragmentObject);、
	然后FragmentObject的onCreateView()方法中返回它自己的页面布局

- 在Fragment中得到Context对象： getActivity()， 这人方法是这样写的：

----------

	final public FragmentActivity getActivity() {
        return mActivity;
    }
	即在fragment的声明周期方法没有被Activity的声明周期调时， 你时得不到Context对象的。
	例如：你在Fragment类声明了一个字段： 
	private AppLockDao dao = new AppLockDao(getActivity);  //你是得不到的
