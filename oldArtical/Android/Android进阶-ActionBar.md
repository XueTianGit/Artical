title: Android进阶-ActionBar与DrawerLayout
date: 2016/2/28 10:31:15 
categories: Android
---

# Android进阶-ActionBar与 DrawerLayout #

>- 先抒发一下蛋疼的心情
>- 接触新事物时，没人指导真的很蛋疼
>- 当初学习AndroidStudio时，搞了好久才能用！ 
>- 这次用它建一个工程，直接报错 values-v23.xml 报错， 又找不到什么东西！
>- nnd！ V7包是你导！你还给我报错， 你到底让我怎么办！！！！， 最烦的就是在这样的地方浪费时间，
>- 用个Genymotion模拟器， 创建模拟器时又连接超时！，网上搜答案，回答都一样！！ 真佩服， 不会就不会你乱复制什么答案！！！
>- 虚拟机装个OSX装了一个星期， 装完又为Xcode烦了好久
>- 天啊！！！！！！学个习！ 为何如此的艰难啊！
>- 好了，牢骚发完了， 默默的继续学习。。。。。。。。。

## ActionBar ##
>- 它是Google在android3.0之后推出的用来设计应用头部标题栏的API
>- A dedicated space for giving your app an identity and indicating the user's location in the app.
>- Access to important actions in a predictable way (such as Search).
>- Support for navigation and view switching (with tabs or drop-down lists).

>- 声明一点的是， 由于它是Android3.0之后推出的， 因此低版本的系统是不能直接使用的
>- 若想使用应引入 v7-appcompat


- 使用步骤
>- 对于3.0以上的APP （大于 <uses-sdk android:minSdkVersion="11" ... />）
>    - Activity的主题应设置为 Theme.Holo 下的主题(自定义的主题Holo应该是他爹)
>- 对于3.0以下的APP
>    - 先导入 v7 appcompat library
>    - Activity 应继承 ActionBarActivity
>    - 主题应为： @style/Theme.AppCompat.Light（同理， 自定义的主题的爹应该是它） 


- 为ActionBar上增加search按钮
>-  在res/menu/下创建一个xml文件， 例如（main_activity_actions.xml）
>-  配置如下
>     - 如果资源文件缺少的话， 应下载 Action Bar Icon Pack

	<menu xmlns:android="http://schemas.android.com/apk/res/android" >
	    <!-- Search, should appear as action button -->
	    <item android:id="@+id/action_search"
	          android:icon="@drawable/ic_action_search"
	          android:title="@string/action_search"
	          android:showAsAction="ifRoom" />
	</menu>

> - 如果这里想要兼容低版本， 应引入 xmlns:yourapp="http://schemas.android.com/apk/res-auto" 名称空间
> - 并将 android:showAsAction="ifRoom" 改为 yourapp:showAsAction="ifRoom"
> - 3. 在代码中做如下处理：
  
	    @Override
		public boolean onCreateOptionsMenu(Menu menu) {
			getMenuInflater().inflate(R.menu.activity_main, menu);
			SearchView searchView = (SearchView) menu.findItem(R.id.action_search)
					.getActionView();
			searchView.setOnQueryTextListener(this);//  搜索的监听
			return true;
		}
		  // 当搜索提交的时候
		@Override
		public boolean onQueryTextSubmit(String query) {
			Toast.makeText(getApplicationContext(), query, 0).show();
			return true;
		}
		// 当搜索的文本发生变化
		@Override
		public boolean onQueryTextChange(String newText) {
			//Toast.makeText(getApplicationContext(), newText, 0).show();
			return true;
		}


- 返回按钮
>- 作用： 当点击ActionBar左上角的返回按钮时，自己finish当前Activity， 并回到来的时候Activity
>- 1. 需要这个功能的Activity的ActionBar应开始这个属性：
	actionBar.setDisplayHomeAsUpEnabled(true);
>- 2.对于，可以回到上一个Activity的Activity，在配置文件中应配置父亲，这样它才知道回到那里去
>    例如：

        <activity
            android:name=".base.SecondActivity"
            android:label="@string/title_activity_second"
            android:parentActivityName=".MainActivity"></activity>  //当点击，actionbar上的返回按钮时，就回到了MainActivity

>- 对于低版本来说 应在<activity/>下配置如下元数据

        <meta-data
            android:name="android.support.PARENT_ACTIVITY"
            android:value="com.itheima.googleplay.MainActivity" />


- 给ActionBar增加标签
>- 通过自定义一个主题， 给主题添加 android:actionBarTabStyle 属性，就可以完成给actionbar增加标签
>- 例如（比忘了，把主题加载需要的activity上）

    <style name="CustomActionBarTheme"
           parent="android:Theme.Light">
        <item name="android:actionBarTabStyle">@style/MyActionBarTabs</item>

        <!-- Support library compatibility -->  
        <item name="actionBarTabStyle">@style/MyActionBarTabs</item>
    </style>

    <!-- ActionBar tabs styles -->
    <style name="MyActionBarTabs"
        parent="android:Widget.ActionBar.TabView">
        <!-- tab indicator -->
        <item name="android:background">@drawable/actionbar_tab_indicator</item>  //actionbar的样式是一个选择器
    </style>

>- 

>- 在代码中可以动态的给这个actionbar添加tab
>- 例如

       //actionbar的tab标签
        actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);
        ActionBar.Tab tab1= actionBar.newTab().setText("标签一").setTabListener(new MyTabListener());
        actionBar.addTab(tab1);

## DrawerLayout ##

>- <android.support.v4.widget.DrawerLayout/>
>- 使用这个控件可以很简单的实现类似与SlidingMenu的效果
>- 抽屉的宽高都是可以自定义的
>- 使用方法
>    - 直接在想要使用这个控件的Activity的布局文件中，将根布局改为如下所示， 即应满足下面的特点
>    - 根布局为 DrawerLayout
>    - 根部局下有一个ViewGroup， 且应匹配父窗体
>    - 再来一个抽屉布局(LL, RL, FL)都可以
>    - 抽屉的方向由 android:layout_gravity="left|right" 决定

	<android.support.v4.widget.DrawerLayout
		xmlns:android="http://schemas.android.com/apk/res/android"
		xmlns:tools="http://schemas.android.com/tools"
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		tools:context=".MainActivity"
		android:id="@+id/dl">
		
		<LinearLayout 
		android:layout_width="match_parent"
		android:layout_height="match_parent">
		<LinearLayout />
	
	    <FrameLayout android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        android:background="@drawable/bg_tab"
	        android:layout_gravity="left">
	
	    </FrameLayout>
	</android.support.v4.widget.DrawerLayout>


## 利用ActionBar来控制抽屉的打开与关闭 ##
>- 我们可以给SlidingMenu增加一个按钮，来控制SlidingMenu的打开与关闭，
>- 利用ActionBar也可以实现这个效果
>- 只需在使用ActionBar的Activity中添加如下代码

        actionBar.setDisplayHomeAsUpEnabled(true);  //增加抽屉控制按钮
   		/*
		 *	1）显示NavigationDrawer的 Activity 对象
			2） DrawerLayout 对象
			3）一个用来指示Navigation Drawer的 drawable资源
			4）一个用来描述打开Navigation Drawer的文本 (用于支持可访问性)。
			5）一个用来描述关闭Navigation Drawer的文本(用于支持可访问性).
		 */
 		DrawerLayout drawerToggle = new ActionBarDrawerToggle(this,
                mDrawerLayout, R.drawable.ic_drawer_am, R.string.open_drawer,
                R.string.close_drawer){

			//两个监听器方法
            @Override
            public void onDrawerClosed(View drawerView) {
                super.onDrawerClosed(drawerView);
                Toast.makeText(getApplicationContext(), "抽屉关闭了", 0).show();
            }
            @Override
            public void onDrawerOpened(View drawerView) {
                super.onDrawerOpened(drawerView);
                Toast.makeText(getApplicationContext(), "抽屉打开了", 0).show();
            }

        };
        mDrawerLayout.setDrawerListener(drawerToggle);
        //  让开关和actionbar建立关系
        drawerToggle.syncState();


>- 并且应在ActionBar对menuitem处理的监听方法中这样写

	@Override
    public boolean onOptionsItemSelected(MenuItem item) {     
        return drawerToggle.onOptionsItemSelected(item)|super.onOptionsItemSelected(item);
    }