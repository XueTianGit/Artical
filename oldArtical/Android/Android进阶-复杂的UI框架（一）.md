title: Android进阶-复杂的UI框架（一）
date: 2016/3/1 9:35:20               
categories: Android
---

# Android进阶-复杂的UI框架（一） #

> 先来看一下， 要构建的这个比较复杂的UI框架的大体构建， 与其所构建的页面
![](http://7xrbxa.com1.z0.glb.clouddn.com/android%E5%A4%8D%E6%9D%82%E7%9A%84UI%E6%A1%86%E6%9E%B6%E4%B8%801.jpg)

![](http://7xrbxa.com1.z0.glb.clouddn.com/android%E5%A4%8D%E6%9D%82%E7%9A%84UI%E6%A1%86%E6%9E%B6%E4%B8%803.png)

![](http://7xrbxa.com1.z0.glb.clouddn.com/android%E5%A4%8D%E6%9D%82%E7%9A%84UI%E6%A1%86%E6%9E%B6%E4%B8%804.png)


- 主页面的构建

![](http://7xrbxa.com1.z0.glb.clouddn.com/android%E5%A4%8D%E6%9D%82%E7%9A%84UI%E6%A1%86%E6%9E%B6%E4%B8%802.5.png)

> - 在MainActivity中使用Fragment: LeftFragment 与 ContentFragment
> - 为了以后数据沟通的方便， 在MainActivity中暴露了两个Fragment的访问方法
	//在绑定Fragment时， 给它增加了Tag	
	fmTransaction.replace(R.id.fgLeftMenu, new LeftFragment(), FG_LEFT_MENU);
	fmTransaction.replace(R.id.fl_content, new ContentFragment(), FG_CONTENT);
	
	 // 获取侧边栏fragment
	    public LeftFragment getLeftMenuFragment() {
	       FragmentManager fm = getFragmentManager();
	        LeftFragment fragment = (LeftFragment) fm
	                .findFragmentByTag(FG_LEFT_MENU);
	
	        return fragment;
	    }
	
	    // 获取主页面fragment
	    public ContentFragment getContentFragment() {
	        FragmentManager fm = getFragmentManager();
	        ContentFragment fragment = (ContentFragment) fm
	                .findFragmentByTag(FG_CONTENT);
	        return fragment;
	    }

- ContentFragment的构建
![](http://7xrbxa.com1.z0.glb.clouddn.com/android%E5%A4%8D%E6%9D%82%E7%9A%84UI%E6%A1%86%E6%9E%B6%E4%B8%802.png)


> - 如图所示， 利用RadioGroup和ViewPager
> - 完成了可用通过点击RadioButton来切换界面的效果
> - 代码中应注意的点都有
>   - 如何设置RadioButton是这种样式
>   - 禁掉ViewPager的滑动事件， 并去除其界面切换的动画效果

	//在布局文件中对RadioButton的样式是这样设置的：
	android:button = "@null"  <!-- 即去除原生的圆圈点击的样子-->
	android:drawableTop="@drawable/btn_tab_home_selector"  <!-- 把图片设置在上面-->
	//对于ViewPager滑动事件的去除方式是通过继承ViewPager，并重写onTouchEcent()方法
	public class NoScrollViewPager extends ViewPager {
		/*.......*/
		/**
		 * 重写onTouchEvent事件,什么都不用做
		 */
		@Override
		public boolean onTouchEvent(MotionEvent arg0) {
			return false;
		}
	}
	//通过对RadioButton的点击实现界面切换，并去除动画效果可以这样实现：
	// 监听RadioGroup的选择事件
		rgGroup.setOnCheckedChangeListener(new OnCheckedChangeListener() {

			@Override
			public void onCheckedChanged(RadioGroup group, int checkedId) {
				switch (checkedId) {
				case R.id.rb_home:
					mViewPager.setCurrentItem(0, false);// false去掉切换页面的动画
					break;
					/*...............................*/
				}
			}
		});
>- 对于ViewPager中的内容，每个RadioButton对应的界面都是不一样的，但是，仔细观察，还是可以抽取出一个
> BasePager，作为5个RadioButton对应界面的基类
>- 这个BasePager中抽取出来的公共布局，是 标题， 内容， 标题上的菜单按钮
	//XML布局， 如下所示
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/title_red_bg" >

        <TextView
            android:id="@+id/tv_title"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerHorizontal="true"
            android:layout_centerVertical="true"
            android:text="智慧北京"
            android:textColor="#fff"
            android:textSize="22sp" />

        <ImageButton
            android:id="@+id/btn_menu"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerVertical="true"
            android:layout_marginLeft="5dp"
            android:background="@null"
            android:src="@drawable/img_menu" />
    </RelativeLayout>

    <FrameLayout
        android:id="@+id/fl_content"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" >
    </FrameLayout>

	</LinearLayout>

	//java代码
	public class BasePager {
		public Activity mActivity;   //便于数据沟通交流
		public View mRootView;// 布局对象
		public TextView tvTitle;// 标题对象
		public FrameLayout flContent;// 内容
		public ImageButton btnMenu;// 菜单按钮
		public BasePager(Activity activity) {
			mActivity = activity;
			initViews();      //这样，每个子类实例，就都会调用它，得到他所需要的控件对象了
		}
	
		/**
		 * 初始化布局
		 */
		public void initViews() {
			mRootView = View.inflate(mActivity, R.layout.base_pager, null);
			tvTitle = (TextView) mRootView.findViewById(R.id.tv_title);
			flContent = (FrameLayout) mRootView.findViewById(R.id.fl_content);
			btnMenu = (ImageButton) mRootView.findViewById(R.id.btn_menu);
		}
		
		/**
		 * 初始化数据
		 */
		public void initData() {
		}
	}


- 主页下面的5个子页面具体内容的构建

![](http://7xrbxa.com1.z0.glb.clouddn.com/android%E5%A4%8D%E6%9D%82%E7%9A%84UI%E6%A1%86%E6%9E%B6%E4%B8%802.5.png)
![](http://7xrbxa.com1.z0.glb.clouddn.com/android%E5%A4%8D%E6%9D%82%E7%9A%84UI%E6%A1%86%E6%9E%B6%E4%B8%804.png)


> - 首先， 所应展示的具体内容，依赖于用户对侧边栏的专题的选择
> - 内容展示的地方，位于上面所构建的主页面的FrameLayout中， 即 public FrameLayout flContent;
> - 由牵扯到侧边栏， 因此这里的数据交流发生在Fragment之间， 如何交流看代码
> - 对于，内容的展示，也是可以进行base页面的抽取的
	//依赖侧边栏，填充具体内容
	//监听，侧边栏，ListView的点击事件
	lvList.setOnItemClickListener(new OnItemClickListener() {

		@Override
		public void onItemClick(AdapterView<?> parent, View view,
				int position, long id) {
			mCurrentPos = position;
			mAdapter.notifyDataSetChanged();
			setCurrentMenuDetailPager(position);  //切换详情页（子页面的具体内容）
		}
	});
	/**
	 * 设置当前菜单详情页， 
	 */
	protected void setCurrentMenuDetailPager(int position) {
		MainActivity mainUi = (MainActivity) mActivity;   //Activity作为Fragment之间交流的中枢
		ContentFragment fragment = mainUi.getContentFragment();// 获取主页面fragment
		NewsCenterPager pager = fragment.getNewsCenterPager();// 获取新闻中心页面
		pager.setCurrentMenuDetailPager(position);// 设置当前菜单详情页
	}

	/**
	 * 设置当前菜单详情页
	 */
	public void setCurrentMenuDetailPager(int position) {
		BaseMenuDetailPager pager = mPagers.get(position);// 获取当前要显示的菜单详情页
		flContent.removeAllViews();// 清除之前的布局， 防止遮盖， 一了百了
		flContent.addView(pager.mRootView);// 将菜单详情页的布局设置给帧布局
		// 设置当前页的标题
		NewsMenuData menuData = mNewsData.data.get(position);
		tvTitle.setText(menuData.title);
		pager.initData();// 初始化当前页面的数据
	}

> 对 BaseMenuDetailPager（详情页的基类）的抽取， 主要抽出了mRootView， 这样有利于详情内容的实现
	public abstract class BaseMenuDetailPager {

		public Activity mActivity;
		public View mRootView;// 根布局对象
		public BaseMenuDetailPager(Activity activity) {
			mActivity = activity;
			mRootView = initViews();
		}	
		/**
		 * 初始化界面
		 */
		public abstract View initViews();
	
		/**
		 * 初始化数据
		 */
		public void initData() {
	
		}
	}

- 对详情页内容的布局
![](http://7xrbxa.com1.z0.glb.clouddn.com/android%E5%A4%8D%E6%9D%82%E7%9A%84UI%E6%A1%86%E6%9E%B6%E4%B8%804.png)

>- 可以看出，详情页的内容布局是： ViewPager 和 ViewPagerIndicator

- 详情页中ViewPager的子View
> 它的里面又包裹了一个ViewPager和ListView

	
	



