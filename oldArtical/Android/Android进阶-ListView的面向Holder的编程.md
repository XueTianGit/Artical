title: Android进阶-ListView的面向Holder的编程
date: 2016/3/3 21:07:33                    
categories: Android
---


# Android进阶-ListView的面向Holder的编程 #

## 核心思想 ##
>- ListView几乎在每一个App中都会用到
>- 对ListView抽取一个框架，将会非常的实用
>- 抽取思想
>    - Adapter负责ListView数据的展现的控制
>    - Holder负责将Adapter给他的数据进行展现
> - 这里面向Holder的编程是指，对于ListView的每一种条目，我们都可以写一个Holder来展现这个条目
>   - 面向Holder的编程，主要是通过Adapter的下面两个方法实现的

	public int getItemViewType(int position) /** 根据位置 判断当前条目是什么类型 */
	public int getViewTypeCount()； /** 当前ListView 有几种不同的条目类型 */

>- 我们可以通过覆写这两个方法，从而实现在ListView展示数据时给不同的Holder去设置数据
>- 并且Android系统，可以根据这两个方法，从而给你维护多个类型的ListView条目的缓存


- 下面的代码，通过对Adapter和Holder的抽取，完成了上面的想法，实现了ListView的面向Holder的编程


>- 首先来看一下这个DefaultAdapter   
>    - 它继承自BaseAdapter
>    - 其中，维护了ListView要展示的数据，并且具有加载更多的功能（可以选择启用与不启用）
>    - 它只是控制把数据交给不同条目的Holder，数据的展示是交给Holder来完成的
>    - 默认包含两种ListView条目，一种是对ListView普通数据展现的条目
>    - 另一种条目是在ListView下滑到最下面时，用于提示用户正在加载更多的条目（可以不启用）   
>    - 其他细节看代码

	public abstract class DefaultAdapter<Data> extends BaseAdapter {
		//维护的数据集合，是普通holder展示的数据
		protected List<Data> datas;
		//两种类型的ListView条目
		private static final int DEFAULT_ITEM = 0;
		private static final int MORE_ITEM = 1;
	
		//在Adapter初始化时，应把要展示的数据，交给adapter
		public DefaultAdapter(List<Data> datas) {
			this.datas = datas;
		}
		
		//获取用于展示普通条目的Holder
		protected abstract BaseHolder<Data> getHolder();
		
		//获取，用于展示“加载更多”的holder
		private MoreHolder holder;
		private BaseHolder getMoreHolder() {
			if(holder!=null){
				return holder;
			}else{
				holder=new MoreHolder(this，hasMore());
				return holder;
			}
		}

		/**
		 * 是否有额外的数据， 即是否使用加载更多的功能（默认有加载更多的功能）
		 */
		protected boolean hasMore() {
			return true;
		}
		
		//用于加载更多数据，具体怎么加载，应由子类实现
		protected  abstract List<Data> onload();
		
		
		@Override
		public int getCount() {
			return datas.size() + 1;    // 最后的一个条目 就是加载更多的条目
		}
		
		@Override
		public long getItemId(int position) {
			return position;
		}
	
		@Override
		public Object getItem(int position) {
			return datas.get(position);
		}
	
		/** 根据位置 判断当前条目是什么类型 */
		@Override
		public int getItemViewType(int position) {  //20     
			if (position == datas.size()) { // 当前是最后一个条目
				return MORE_ITEM;
			}
			return DEFAULT_ITEM; // 如果不是最后一个条目 返回默认类型
		}
	
		/** 当前ListView 有几种不同的条目类型 */
		@Override
		public int getViewTypeCount() {
			return super.getViewTypeCount() + 1; // 2 有两种不同的类型
		}
	
	
		/*
		 *android系统会根据 getItemViewType()，返回的结果来给我们维护对应数量的convertView
		 *从而，优化了ListView的数据展示
		 * */
		public View getView(int position, View convertView, ViewGroup parent) {
			
			BaseHolder holder = null;
			
			switch (getItemViewType(position)) {  // 判断当前条目时什么类型
			case MORE_ITEM:     
				if(convertView==null){
					holder=getMoreHolder();
				}else{     
					holder=(BaseHolder) convertView.getTag();
				}
				break;
				
			case DEFAULT_ITEM:
				if (convertView == null) {
					holder = getHolder();
				} else {
					holder = (BaseHolder) convertView.getTag();
				}
				if (position < datas.size()) {
					holder.setData(datas.get(position));
				}
				break;
			}
			
			//返回的View根据不同的Holder返回不同的View
			return holder.getConvertView();  //  如果当前Holder 恰好是MoreHolder  证明MoreHOlder已经显示
		}
		
		
		/**
		 * 当加载更多条目显示的时候 调用该方法
		 */
		public void loadMore() {
			ThreadManager.getInstance().createLongPool().execute(new Runnable() {			
				@Override
				public void run() {
					// 在子线程中加载更多 
					final List<Data> newData = onload(); //加载更多的数据，并交给Holder来展示
					
					UiUtils.runOnUiThread(new Runnable() {					
						@Override
						public void run() {
							if(newData==null){
								holder.setData(MoreHolder.LOAD_ERROR);//  
							}else if(newData.size()==0){
								holder.setData(MoreHolder.HAS_NO_MORE);
							}else{
								// 成功了
								holder.setData(MoreHolder.HAS_MORE);
								datas.addAll(newData);//  给listView之前的集合添加一个新的集合
								notifyDataSetChanged();// 刷新界面						
							}
							
						}
					});			
				}
			});
		}
	
		public List<Data> getDatas() {
			return datas;
		}
	
		public void setDatas(List<Data> datas) {
			this.datas = datas;
		}
	
	}

- 看完上面的代码，可能还是不理解，没关系只看一个肯定是不能理解的，下面来看一下BaseHolder，他是怎么完成数据展示的
>- 它维护了ListView对条目的缓存对象 convertView
>- ListView的Adapter，可以把要展示的数据设置给BaseHolder
>- 如何展示数据，交个之类去实现

	public abstract class BaseHolder<Data>  {
		private View convertView;   //ListView的缓存View对象， 
		private Data data;          //ListView交给他展示的数据
		protected BitmapUtils bitmapUtils;   //一般的ListView可能都有图片
		public BaseHolder(){
			bitmapUtils = BitmapHelper.getBitmapUtils();
			convertView=initView();    //convertView缓存什么样的View，具体由Holder的实现类巨鼎
			convertView.setTag(this);    //把这个Holder交给contentView， 以便在ListView中获取
		}

		public View getConvertView() {   //方便ListView获取convertView
			return convertView;
		}
		public void setData(Data data){
			this.data=data;
			refreshView(data);
		}

		/** 创建界面*/
		public  abstract View initView();
		/** 
		 * 根据数据刷新界面
		 * 具体怎么刷新，子类去想吧
		 * */  
		public abstract void refreshView(Data data);     
	}


- 下面就来看一下具体是怎么应用， 与实现面向Holder的编程的

----------

	// 展示那种数据? 如何加载数据，数据交给哪个Holder去展示
	public abstract class ListBaseAdapter extends DefaultAdapter<AppInfo> {
		public ListBaseAdapter(List<AppInfo> datas) {
			super(datas);
		}
	
		@Override
		protected BaseHolder<AppInfo> getHolder() {
			return new ListBaseHolder();
		}
	
		@Override
		protected abstract List<AppInfo> onload(){
			//加载数据
			HomeProtocol protocol=new HomeProtocol();
			List<AppInfo> load = protocol.load(datas.size());
			return load；
		}
	}

	//定义要展示数据的条目，数据如何展示到条目上
	public class ListBaseHolder extends BaseHolder<AppInfo> {
		//用于展示条目数据的控件
		ImageView item_icon;
		TextView item_title,item_size,item_bottom;
		RatingBar item_rating;
		//将数据展示到条目的具体方法
		public void refreshView(AppInfo data){
			this.item_title.setText(data.getName());// 设置应用程序的名字
			String size=Formatter.formatFileSize(UiUtils.getContext(), data.getSize());
			this.item_size.setText(size);
			this.item_bottom.setText(data.getDes());
			float stars = data.getStars();
			this.item_rating.setRating(stars); // 设置ratingBar的值
			String iconUrl = data.getIconUrl();  //http://127.0.0.1:8090/image?name=app/com.youyuan.yyhl/icon.jpg
			
			// 显示图片的控件
			bitmapUtils.display(this.item_icon, HttpHelper.URL+"image?name="+iconUrl);
		}
	
		//返回Adapter缓存的ConvertView
		@Override
		public View initView() {
			View convertView=View.inflate(UiUtils.getContext(), R.layout.item_app, null);
			this.item_icon=(ImageView) convertView.findViewById(R.id.item_icon);
			this.item_title=(TextView) convertView.findViewById(R.id.item_title);
			this.item_size=(TextView) convertView.findViewById(R.id.item_size);
			this.item_bottom=(TextView) convertView.findViewById(R.id.item_bottom);
			this.item_rating=(RatingBar) convertView.findViewById(R.id.item_rating);
			return convertView;
		}
	}
	
	//加载更多Holder的实现
	public class MoreHolder extends BaseHolder<Integer> {
		public static final int HAS_NO_MORE=0;  // 没有额外数据了
		public static final int LOAD_ERROR=1;// 加载失败
		public static final int HAS_MORE=2;//  有额外数据
		private boolean  hasMore;   //是否还有更多的数据
		private RelativeLayout rl_more_loading,rl_more_error;  //
		
		/**当Holder显示的时候 显示什么样子*/
		@Override
		public View initView() {
			View view=UiUtils.inflate(R.layout.load_more);
			rl_more_loading=(RelativeLayout) view.findViewById(R.id.rl_more_loading);
			rl_more_error=(RelativeLayout) view.findViewById(R.id.rl_more_error);
			return view;
		}

		private DefaultAdapter adapter;  //
		public MoreHolder(DefaultAdapter adapter，boolean hasMore) {
			super();
			this.adapter=adapter;
			this.hasMore=hasMore;
			if(!hasMore){   //没有更多数据
				setData(0);   //会引起 refreshView()的调用， 即不会显示加载hasMore的选项了
			}		
		}
	
	
		@Override
		public View getConvertView() {
			loadMore();	   //只要看到这个ListView条目，就要去加载数据
			return super.getConvertView();
		}
	
		private void loadMore() {
			// 请求服务器   加载下一批数据 
			//  交给Adapter  让Adapter  加载更多数据， 我只负责展示数据
			adapter.loadMore();  
		}

		/**根据数据做界面的修改*/
		@Override
		public void refreshView(Integer data) {
			rl_more_error.setVisibility(data==LOAD_ERROR?View.VISIBLE:View.GONE);
			rl_more_loading.setVisibility(data==HAS_MORE?View.VISIBLE:View.GONE);
		}


- 上面就完成了一个带有加载更多功能的DefaultAdapter的抽取
- 那么，如何是面向Holder的编程呢？
- 看下面这个工程的实例
- 需求： 给ListView的部加一个条目， 用来展示一个ViewPager

>- 首先来个一下用于展示含有ViewPager的这个条目的Holder如何写的

	public class HomePictureHolder extends BaseHolder<List<String>> {
		/*当new HomePictureHolder()就会调用该方法 */
		private ViewPager viewPager;
		private List<String> datas;
		@Override
		public View initView() {
			/*在这里， 通过填充布局的方法， 获得用于展示这个条目的布局，并交给ListView去缓存了*/
			return thisView;
		}

		/*当 holder.setData 才会调用*/
		@Override
		public void refreshView(List<String> datas) {
			this.datas=datas;
			viewPager.setAdapter(new HomeAdapter());
		}
		
		class HomeAdapter extends PagerAdapter{
			// 当前viewPager里面有多少个条目
			@Override
			public int getCount() {
				return datas.size();
			}
			/*判断返回的对象和 加载view对象的关系*/
			@Override
			public boolean isViewFromObject(View arg0, Object arg1) {
				return arg0==arg1;
			}
	
			@Override
			public void destroyItem(ViewGroup container, int position, Object object) {
				container.removeView((View)object);
			}
	
			@Override
			public Object instantiateItem(ViewGroup container, int position) {
				ImageView imageView=new ImageView(UiUtils.getContext());
				bitmapUtils.display(imageView, HttpHelper.URL+"image?name="+datas.get(position));
				container.addView(imageView);  //加载的view对象
				return imageView; // 返回的对象
			}
			
		}
	
	}


>- 在客户端， 是这样把这个Holder加到ListView中

		HomePictureHolder holder=new HomePictureHolder();
		holder.setData(pictures);
		View contentView = holder.getContentView(); // 得到holder里面管理的view对象
		listView.addHeaderView(contentView); // 把holder里的view对象 添加到listView的上面	
		listView.setAdapter(new ListBaseAdapter(datas));


- 因此，以后对于，想要往ListView中加一个特殊的条目，只需要写一个Holder就可以了