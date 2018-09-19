title: Android项目-闪屏界面与GridView
date: 2016/3/7 18:27:10      
categories: Android
---

# Android项目-闪屏界面与GridView#

## 闪屏界面 ##

一般闪屏界面是应用的第一个界面。应用在闪屏界面做的主要工作一般有：

- 展示logo（应用logo、公司logo）
- 项目初始化
- 检测版本更新
- 校验程序的合法性（比如是否有网络）

> 闪屏界面显示的内容一般都是一张背景图片。我们将图片作为布局的背景即可
> 闪屏页面一般至少要显示一段时间

- Point1 （去除标题）

一般应用，是没有标题的，有的话太难看了。那标题如何去除呢？
方法主要是更改主题，android中有无标题的主题，那么怎么设置：

	1. 可以直接引用这个主题
	 <application
        android:theme="@android:style/Theme.Light.NoTitleBar.Fullscreen">
	2. 我们可以给 @style/AppTheme 添加隐藏标题这个样式
		<style name="AppTheme" parent="AppBaseTheme">
			<!-- 隐藏标题栏 -->
			<item name="android:windowNoTitle">true</item>
		</style>

- Point2（给文字设置阴影）

跟文字设置阴影可以使文字看起来更有立体感，设置阴影主要依赖下面4个属性：

    <TextView
        android:shadowColor="#f00"
        android:shadowDx="1"
        android:shadowDy="1"
        android:shadowRadius="1"
        android:text="我有阴影" />
可以看出设置阴影，其实就是使文字进行了偏移。

- Point3（动态获取版本信息）

主要依赖PackageManager：

		PackageManager packageManager = getPackageManager();
		PackageInfo packageInfo = packageManager.getPackageInfo(getPackageName(), 0);
		mVersionCode = packageInfo.versionCode;
		mVersionName = packageInfo.versionName;

- Point4 （版本更新）

一般步骤为：
1. 获取服务器的版本信息（一般以JSON格式交互）
2. 根据版本信息提示用户是否下载新版本，并更新。

相关知识和应注意的点有：	
	 
	- 获取网络权限      android.permission.INTERNET
	- HttpURLConnection, URL
	- 在模拟器下访问本机Tomcat， 地址可使用10.0.2.2 或者 你的网络的真实IP
	- Android下解析JSON的对象： JSONObject
	- 对话框 AlertDialog.Builder AlertDialog
	- Activity跳转时， 思考该Activity是不是要finish()

- Point5 （下载APK文件）

在上面， 如果用户选择更新， 那么就需要下载新版本的APK。
相关知识和应注意的点有：
	
	- 文件的下载， 使用的是第三方框架XUtils
	- 下载文件到本地时， 应先判断SD卡是否挂载。和是否已经获得访问SD卡权限 .WRITE_EXTERNAL_STORAGE
    - 下载进度条设置，应下载时才出现， 相关属性： visibility: gone or invisible
    - 下载完成后， 如果在模拟器下，应放在签名文件的冲突
          签名冲突即：两个应用程序，包名冲突，但是签名文件不同；
		  正式签名：有效期长（好几十年）， 需设置密码，发布应用时，应使用它来打包
		  测试签名：有效期短（1年），eclipse默认签名为debug.keystore， 别名为：android
				   密码为:androiddebugkey
	- 现在完成后，应启动安装界面：
		Intent intent = new Intent(Intent.ACTION_VIEW);
		intent.addCategory(Intent.CATEGORY_DEFAULT);
		intent.setDataAndType(Uri.fromFile(fileInfo.result),
				"application/vnd.android.package-archive");
		startActivityForResult(intent, 0);
	-  这样启动，是因为，如果用户不想安装， 我们可以再回调函数onActivityResult()中，继续做事

- Point6 （TextView强制获取焦点）

> 如果我们想使TextView显示的内容，类似走马灯似的移动的话也是可以的， 不过前提是TextView得获得焦点
> 可以这样设置：


    <TextView
        android:ellipsize="marquee"
        android:focusable="true"
        android:focusableInTouchMode="true"
        android:singleLine="true"
        android:text="我会循环显示，哈哈哈哈哈！！！"
	/>


- Point7 (细节处理)
	- 设置对话框的取消监听，builder.setOnCancelListener（）
	- eclipse下对应用打包  
		AndroidTools->export signed application package ->
	- 对于UI的刷新， 我们可以再Handler中，统一处理。
	- 我们可以给闪屏页面添加渐变动画
		//闪屏页的渐变动画
		AlphaAnimation alphaAnimation = new AlphaAnimation(0.2f, 1f);
		alphaAnimation.setDuration(2000);
		rl.startAnimation(alphaAnimation);   //rl为闪屏页所属的线性布局

## GridView ##

> 对于条状的显示内容，我们可以使用ListView, 而对于类似九宫格似的样式， 我们则可以使用
> GridView， 对于GridView的使用几乎和ListView一模一样

使用步骤：

-  在布局文件中， 添加GridView组件

例如，将下面这个GridView设置为每行3列， 占满屏幕

	<GridView
        android:id="@+id/gv_home"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:numColumns="3"
        android:verticalSpacing="20dp" >
	</GridView>
- 设计填充GridView元素的样式
- 在代码中填充布局文件中的GridView
		setContentView(R.layout.activity_home);
		gvHome = (GridView) findViewById(R.id.gv_home);
		gvHome.setAdapter(new HomeAdapter());

		class HomeAdapter extends BaseAdapter{
	
			@Override
			public int getCount() {
				return mItems.length;
			}
	
			@Override
			public Object getItem(int position) {
				
				return mItems[position];
			}
	
			@Override
			public long getItemId(int position) {
				
				return position;
			}
	
			@Override
			public View getView(int position, View convertView, ViewGroup parent) {
				View gvItem = View.inflate(HomeActivity.this, R.layout.grid_item, null);
				TextView tv = (TextView) gvItem.findViewById(R.id.tv_item);
				ImageView iv = (ImageView) gvItem.findViewById(R.id.iv_item);
				tv.setText(mItems[position]);
				iv.setImageResource(mPics[position]);
				
				return gvItem;
			}
			
		}
	
	



	

 


		

    	

		
		
	

