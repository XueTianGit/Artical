title: Android进阶-自定义控件一
date: 2016/3/1 9:15:21              
categories: Android
---

# Android进阶-自定义控件一 #

- 自定义控件的分类
	- 组合控件：将系统原生控件组合起来，加上动画效果，形成一种特殊的UI特效
	- 纯粹自定义控件：继承系统View，自己去实现View效果

- 旋转动画的注意点
> - 当x,y坐标相对于自己时，x与y的大小为0-1；
> - RotateAnimation.setFillAfter(true); 使动画结束后保持结束状态
> - RotateAnimation.setStartOffset()；  用来设置动画的延时开启时间
> - 系统原生的动画，并不会改变View位置，因此在动画消失后，应注意其其中所包含的点击事件，
>   为了使这些点击事件不再响应，可以当动画消失后，就把点击事件禁掉-> setEnabled(false);  
>   setClickable(false)是不能禁掉的！
> - 对于手机键盘的监听可使用Activity的
>    	public boolean onKeyDown(int keyCode, KeyEvent event) {}
> - 为了使动画完全播放，我们可以监听动画的状态

	static class MyAnimationListener implements AnimationListener{
		@Override
		public void onAnimationStart(Animation animation) {
		}
		@Override
		public void onAnimationEnd(Animation animation) {
		}
		@Override
		public void onAnimationRepeat(Animation animation) {
		}
		
	}


- ViewPager
> - ViewPager是Android3.0之后才出现的
> - 低版本想使用的话应:    <android.support.v4.view.ViewPager>
> - ViewPage类似与ListView， 它展示内容也需要设置ViewPager.setAdapter()

	- ViewPager的Adapter
 	class MyPagerAdapter extends PagerAdapter{

        /**
         * 返回多少page
         */
        @Override
        public int getCount() {
            return 100;
        }

        /**
         * true: 表示不去创建，使用缓存  false:去重新创建
         * view： 当前滑动的view
         * object：将要进入的新创建的view，由instantiateItem方法创建
         */
        @Override
        public boolean isViewFromObject(View view, Object object) {
            return view==object;
        }

        /**
		
				- ViewPager的预加载机制
				- ViewPager在他的缓存中最多保存3个界面
         * 类似于BaseAdapger的getView方法
         * 用了将数据设置给view
         * 由于它最多就3个界面，不需要viewHolder
         */
        @Override
        public Object instantiateItem(ViewGroup container, int position) {
            /* 将ViewPager所将要展示的界面给描绘出来，即创建view对象，并填充数据
				将view对象加到container容器中， 并返回*/
            return view;
        }

        /**
         * 销毁page
         * position： 当前需要消耗第几个page
         * object:当前需要消耗的page
         */
        @Override
        public void destroyItem(ViewGroup container, int position, Object object) {
	//			super.destroyItem(container, position, object);
            container.removeView((View) object);
        }

    }


- 如何获得ViewPager当前所展现的页面位置（position）

viewPager.setOnPageChangeListener()：

	  private void initListener() {
	    viewPager.setOnPageChangeListener(new OnPageChangeListener() {
	        @Override
	        public void onPageSelected(int position) {  
	           	/*可以根据位置position来处理页面*/
	        }
	        @Override
	        public void onPageScrolled(int position, float positionOffset,
	                                   int positionOffsetPixels) {
	        }
	        @Override
	        public void onPageScrollStateChanged(int state) {
	        }
	    });
	}

也可以：
	int currentPage = viewPager.getCurrentItem()；  得到

- 对于selector, 控制图片的显示状态，不仅可以通过按下与松开， 还可以通过设置

----------

	android:state_enabled="true"

	<selector xmlns:android="http://schemas.android.com/apk/res/android" >
	    <item android:state_enabled="true" android:drawable="@drawable/dot_focus"></item>
		<item android:drawable="@drawable/dot_unfocus"></item>
	</selector>
	
	//然后在代码中，动态控制状态
	view.setEnabled(true); // 

- ViewPager的页面循环
	- 伪无限循环
		- 设置        public int getCount() { return Integer.MAX_VALUE;}
		- 在显示页面时， 对position进行处理后再设置页面内容， 例如  currentPage = position % lists.size() //lists为我们要展示数据的集合
		- 上面两步只能完成向右的无限循环，对于向左的无限循环，解决方案是：把开始页码设置在中间
			- 即在初始化数据时： viewPager.setCurrentItem((Integer.MAX_VALUE/5)*5);

- ViewPager的自动切换
	- 使用Handler来循环发送延迟消息， 并在消息中  viewPager.setCurrentItem(viewPager.getCurrentItem() + 1);

- 代码中的宽高设置与布局文件中的宽高设置
	- 在代码中对宽高的指定都是以像素为单位的。
	- 在布局文件中我们可以给宽高指定单位，一般都使用像素
	- 所以代码与布局文件中的宽高值如果相同的话，显示效果可能是不一样的
	
- PopupWindow的细节
	- 在设定PopupWindow的布局位置时， 可以直接将他设置在一个控件的下面： popupWindow.showAsDropDown(View, x , y); 
	- 一般PopupWindow必设的3个属性
		- popupWindow.setFocuable(true)
		- popupWindow.setBackgroundDrawable();  //要想使PopupWindow显示动画，必须要有背景， 背景图片可以不指定，但必须设
		- popupWindow.setOutSideFocusable()  //在popupWindow的外部点击时，可以关闭PopupWindow
		

- 还是ListView条目焦点被抢的问题
	- ListView的焦点会被一些强制获得焦点的控件给抢走，从而不能响应点击事件，还可以在布局文件中这样解决
		- 在listView的根布局上加上:  descendantFocusablity = "blockDescendants"


