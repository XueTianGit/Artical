title: Android进阶-面向Holder的编程
date: 2016/3/3 21:05:23                   
categories: Android
---

# Android进阶-面向Holder的编程 #

- 何为面向Holder的编程
>- 一般我们在开发一个界面时， 一个界面都是可以分为好几个部分的。
>- 界面肯定是用来展示数据的
>- 面向Holder的编程，可以使我们对一个界面的不同部分分别进行开发，即一个部分写一个Holder
>- 面向Hodler编程主要时
>    - 写一个展示界面的Holder
>    - 将Holder添加到中的界面中
>    - 最后多个Holder就可以构成一个界面



- BaseHolder
>- 用来抽取所有Holder公有代码的类， 并制定Holder的规范

	public abstract class BaseHolder<Data>  {
		private View convertView;   //这个Holder所负责的展现的界面View 
		private Data data;          //界面所需要的数据
		protected BitmapUtils bitmapUtils;   //一般界面都要展示图片， 持有一个BitmapUtils不为过
		public BaseHolder(){
			bitmapUtils = BitmapHelper.getBitmapUtils();
			convertView=initView();        //初始化所要展现的界面
			convertView.setTag(this);    //把这个Holder交给contentView， 以便获取
		}

		/** 创建界面， 具体由业务逻辑决定*/
		public  abstract View initView();   
		/** 
		 * 根据数据刷新界面
		 * 具体怎么刷新，子类去想吧
		 * */  
		public abstract void refreshView(Data data);  

		//给Holder设置数据， 设置数据时就要刷新界面
		public void setData(Data data){
			this.data=data;
			refreshView(data);
		}
  		//方便获取convertView
		public View getConvertView() {   
			return convertView;
		} 
	}


- 那么如何应用呢？
>- 既然要把开发的Holder添加的布局中， 那么得先准备好一个布局
>- 这里以帧布局为例

	<FrameLayout
	    android:id="@+id/bottom_layout"
	    android:layout_width="match_parent"
	    android:layout_height="50dp"
	    android:layout_alignParentBottom="true"
	    android:background="@drawable/detail_bottom_bg" >
	</FrameLayout>

>- 下面开发一个Holder
>- 类似下面的

	public class DetailBottomHolder extends BaseHolder<AppInfo> implements OnClickListener {
		@ViewInject(R.id.bottom_favorites)
		Button bottom_favorites;
		@ViewInject(R.id.bottom_share)
		Button bottom_share;
		@ViewInject(R.id.progress_btn)
		Button progress_btn;
		@Override
		public View initView() {   //该Holder展示的界面
			View view=UiUtils.inflate(R.layout.detail_bottom);
			ViewUtils.inject(this, view);
			return view;
		}
	
		@Override 
		public void refreshView(AppInfo data) {  //给Holder刷新数据
			bottom_favorites.setOnClickListener(this);
			bottom_share.setOnClickListener(this);
			progress_btn.setOnClickListener(this);
		}
	
		@Override
		public void onClick(View v) {
			switch (v.getId()) {
			case R.id.bottom_favorites:
				Toast.makeText(UiUtils.getContext(), "收藏", 0).show();
				break;
			case R.id.bottom_share:
				Toast.makeText(UiUtils.getContext(), "分享", 0).show();
				break;
			case R.id.progress_btn:
				Toast.makeText(UiUtils.getContext(), "下载", 0).show();
				break;
			}
		}
	
	}

>- 将开发的Holder，添加到界面中
>- 例如，如下代码

		//示例，
		bottom_layout=(FrameLayout) view.findViewById(R.id.bottom_layout);
		bottomHolder=new DetailBottomHolder();
		bottomHolder.setData(data);
		bottom_layout.addView(bottomHolder.getContentView());