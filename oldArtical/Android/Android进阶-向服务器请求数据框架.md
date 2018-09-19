title: Android进阶-向服务器请求数据框架
date: 2016/3/3 21:05:01                  
categories: Android
---

# Android进阶-向服务器请求数据框架 #

- LoadingPager

----------

	/**
	 * 一个用于加载服务器数据的帧布局
	 * 1. 根据不同的加载状态，可以显示不同的加载界面
	 * 2. 当加载失败时， 可以点击进行重新加载
	 * 3.外部调用时使用show方法，来进行页面的展示
	 * 4.必须实现自己的加载成功的界面
	 * 
	 */
	public abstract class LoadingPage extends FrameLayout {
	
		public static final int STATE_UNKOWN = 0;
		public static final int STATE_LOADING = 1;
		public static final int STATE_ERROR = 2;
		public static final int STATE_EMPTY = 3;
		public static final int STATE_SUCCESS = 4;
		public int state = STATE_UNKOWN;
	
		private View loadingView;// 加载中的界面
		private View errorView;// 错误界面
		private View emptyView;// 空界面
		private View successView;// 加载成功的界面
	
		public LoadingPage(Context context) {
			super(context);
			init();
		}
	
		public LoadingPage(Context context, AttributeSet attrs, int defStyle) {
			super(context, attrs, defStyle);
			init();
		}
	
		public LoadingPage(Context context, AttributeSet attrs) {
			super(context, attrs);
			init();
		}
	
		private void init() {
			loadingView = createLoadingView(); // 创建了加载中的界面
			if (loadingView != null) {
				this.addView(loadingView, new FrameLayout.LayoutParams(
						LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT));
			}
			errorView = createErrorView(); // 加载错误界面
			if (errorView != null) {
				this.addView(errorView, new FrameLayout.LayoutParams(
						LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT));
			}
			emptyView = createEmptyView(); // 加载空的界面
			if (emptyView != null) {
				this.addView(emptyView, new FrameLayout.LayoutParams(
						LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT));
			}
			showPage();// 根据不同的状态显示不同的界面
		}
	
		// 根据不同的状态显示不同的界面
		private void showPage() {
			if (loadingView != null) {
				loadingView.setVisibility(state == STATE_UNKOWN
						|| state == STATE_LOADING ? View.VISIBLE : View.INVISIBLE);
			}
			if (errorView != null) {
				errorView.setVisibility(state == STATE_ERROR ? View.VISIBLE
						: View.INVISIBLE);
			}
			if (emptyView != null) {
				emptyView.setVisibility(state == STATE_EMPTY ? View.VISIBLE
						: View.INVISIBLE);
			}
			
			if (state == STATE_SUCCESS) {
				if (successView == null) {
					successView = createSuccessView();
					this.addView(successView, new FrameLayout.LayoutParams(
							LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT));
				}
				successView.setVisibility(View.VISIBLE);
			} else {
				if (successView != null) {
					successView.setVisibility(View.INVISIBLE);
				}
			}
		}
	
		/* 创建了空的界面 */
		private View createEmptyView() {
			View view = View.inflate(UiUtils.getContext(), R.layout.loadpage_empty,
					null);
			return view;
		}
	
		/* 创建了错误界面 */
		private View createErrorView() {
			View view = View.inflate(UiUtils.getContext(), R.layout.loadpage_error,
					null);
			Button page_bt = (Button) view.findViewById(R.id.page_bt);
			page_bt.setOnClickListener(new OnClickListener() {
	
				@Override
				public void onClick(View v) {
					show();   //重新加载
				}
			});
			return view;
		}
	
		/* 创建加载中的界面 */
		private View createLoadingView() {
			View view = View.inflate(UiUtils.getContext(),
					R.layout.loadpage_loading, null);
			return view;
		}
	
		public enum LoadResult {
			error(2), empty(3), success(4);
	
			int value;
			LoadResult(int value) {
				this.value = value;
			}
			public int getValue() {
				return value;
			}
	
		}
	
		// 根据服务器的数据 切换状态
		public void show() {
			if (state == STATE_ERROR || state == STATE_EMPTY) {
				state = STATE_LOADING;
			}
			// 请求服务器 获取服务器上数据 进行判断
			// 请求服务器 返回一个结果
			ThreadManager.getInstance().createLongPool().execute(new Runnable() {		
				@Override
				public void run() {
					SystemClock.sleep(2000);
					final LoadResult result = load();
					UiUtils.runOnUiThread(new Runnable() {
	
						@Override
						public void run() {
							if (result != null) {
								state = result.getValue();
								showPage(); // 状态改变了,重新判断当前应该显示哪个界面
							}
						}
					});
				}
			});	
			showPage();
		}
	
		/***
		 * 创建成功的界面
		 * 
		 * @return
		 */
		public abstract View createSuccessView();
	
		/**
		 * 请求服务器
		 * 
		 * @return
		 */
		protected abstract LoadResult load();
	}


- 用于向服务器请求数据的类

----------



	public abstract class BaseProtocol<T> {
		
		public T load(int index) {
			SystemClock.sleep(1000);
			// 加载本地数据
			String json = loadLocal(index);
			if (json == null) {
				// 请求服务器
				json = loadServer(index);
				if (json != null) {
					saveLocal(json, index);
				}
			}
			if (json != null) {
				return paserJson(json);
			} else {
				return null;
			}
		}
	
	
		private String loadServer(int index) {
			HttpResult httpResult = HttpHelper.get(HttpHelper.URL +getKey()
					+ "?index=" + index); 
			String json = httpResult.getString();
			return json;
		}
		
		private void saveLocal(String json, int index) {
			
			BufferedWriter bw = null;
			try {
				File dir=FileUtils.getCacheDir();
				//在第一行写一个过期时间 
				File file = new File(dir, getKey()+"_" + index); // /mnt/sdcard/googlePlay/cache/home_0
				FileWriter fw = new FileWriter(file);
				 bw = new BufferedWriter(fw);
				bw.write(System.currentTimeMillis() + 1000 * 100 + "");
				bw.newLine();// 换行
				bw.write(json);// 把整个json文件保存起来
				bw.flush();
				bw.close();
			} catch (Exception e) {
				e.printStackTrace();
			}finally{
				IOUtils.closeQuietly(bw);
			}
		}
	
		private String loadLocal(int index) {
			//  如果发现文件已经过期了 就不要再去复用缓存了
			File dir=FileUtils.getCacheDir();// 获取缓存所在的文件夹
			File file = new File(dir, getKey()+"_" + index); 
			try {
				FileReader fr=new FileReader(file);
				BufferedReader br=new BufferedReader(fr);
				long outOfDate = Long.parseLong(br.readLine());
				if(System.currentTimeMillis()>outOfDate){
					return null;
				}else{
					String str=null;
					StringWriter sw=new StringWriter();
					while((str=br.readLine())!=null){
					
						sw.write(str);
					}
					return sw.toString();
				}
				
			} catch (Exception e) {
				e.printStackTrace();
				return null;
			}
		}
	
		/**
		 * 解析json
		 * @param json
		 * @return
		 */
		public abstract T paserJson(String json);
		/**
		 * 说明了关键字
		 * @return
		 */
		public abstract String getKey();
	}

	//一个实现类的范例
	public class SubjectProtocol extends BaseProtocol<List<SubjectInfo>>{
	
		@Override
		public List<SubjectInfo> paserJson(String json) {
			List<SubjectInfo> subjectInfos=new ArrayList<SubjectInfo>();
			try {
				JSONArray jsonArray=new JSONArray(json);
				for(int i=0;i<jsonArray.length();i++){
					JSONObject jsonObject = jsonArray.getJSONObject(i);
					String des=jsonObject.getString("des");
					String url = jsonObject.getString("url");
					SubjectInfo info=new SubjectInfo(des, url);
					subjectInfos.add(info);
					
				}
				return subjectInfos;
				
			} catch (JSONException e) {
				e.printStackTrace();
				return null;
			}
		}
	
		@Override
		public String getKey() {
			return "subject";
		}
	
	
	}