title: Android项目-分页与分批及XUtils的注解来代替findViewById
date: 2016/2/29 15:41:48  
categories: Android
---

# Android项目-分页与分批及XUtils的注解来代替findViewById#

- Point1 使用ListView完成分页与分批
	- 核心思想当然都是，获取数据，在ListView中展示
	- 涉及的数据库语句：    SELECT * FROM INFOS LIMIT ? OFFSET ?
	- 但区别是：
		- 分页的数据是在改变的
		- 分批的数据是不断增加的


> 例如 List<Infoinfos; infos中放有我们要展示的数据，
> 那么对于分页，我们可能会这样处理：

	//响应用户改变页面的交互代码
	public void changePage(View view){
		/*处理页码，填充数据...*/
        changeData();
    }

	//当用户获取下一页数据， 这样更新infos中的数据， 这里infos是全局变量
	private void changeData() {
		infos = dao.findPart(curentPgeNumber, pageSize);
		handler.sendEmptyMessage(0);
	}
	//在Handler中，这样显示新的数据
	private Handler handler = new Handler(){
		public void handleMessage(android.os.Message msg) {
				lvPage.setAdapter(new MyPageAdapter());  
			}
		};
	};

> 但是对于分批，我们可能这样处理：

	//监听ListView的滑动，以确定是否加载数据
	lvBull.setOnScrollListener(new OnScrollListener() {
			// 滚动状态发生变化调用的方法。
			// OnScrollListener.SCROLL_STATE_FLING 惯性滑动
			// OnScrollListener.SCROLL_STATE_TOUCH_SCROLL 触摸滑动
			// OnScrollListener.SCROLL_STATE_IDLE 静止
			@Override
			public void onScrollStateChanged(AbsListView view, int scrollState) {
				switch (scrollState) {
				case OnScrollListener.SCROLL_STATE_IDLE: // 静止状态
					// 判断是否是最后一个条目。
					int lastPosition = lv_callsms_safe.getLastVisiblePosition();
					System.out.println("最后一个可见条目的位置：" + lastPosition);
					if (lastPosition == infos.size() - 1) { //最后一个位置，是否为能展示的最后一条数据
						/*.......*/
						loadData();
					}
					break;
				}
			}

	private void loadData() {
		infos.addAll(dao.findPart2(startIndex, maxCount));//将新加载的数据添加到数据集合中
		handler.sendEmptyMessage(0);
	}

	private Handler handler = new Handler() {
		public void handleMessage(android.os.Message msg) {
			adapter.notifyDataSetChanged(); //通知adapter，展示数据已经改变
		}
	};


- Point2 使用XUtils的注解方式来代替findViewById

----------

	1. 在onCreate()方法中，开启这个操作

		@Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
			/*....*/
	        ViewUtils.inject(this);
	    }

	2. 在需要findViewById的控件上，添加@ViewInject注解，例如
	    @ViewInject(R.id.btnAddBlackNumber)
   		private Button btnAddBlackNumber;
	
	
		


