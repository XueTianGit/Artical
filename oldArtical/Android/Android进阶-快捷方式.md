title: Android项目-快捷方式
date: 2016/2/29 15:42:57    
categories: Android
---

# Android项目-快捷方式 #
如何在桌面上创建一个快捷方式呢？
>- 桌面也是一个App，要想在桌面上创建一个快捷方式得靠他
>- 在Android的系统应用程序Launcher2中提供了一个广播接收者：InstallShortcutReceiver
>- 我们可以通过给他发送一个安装快捷方式的广播，来安装我们的App的快捷方式

	/*
	 *创建一个可以快速打电话的快捷方式
	 */
	public class CallShortCutActivity extends Activity {
	
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			
			Intent intent = new Intent();
			intent.setAction("com.android.launcher.action.INSTALL_SHORTCUT");  //
			
			//dowhtIntent 用于告诉InstallShortcutReceiver， 我们要创建的快捷方式的样子
			Intent dowhtIntent = new Intent();
			dowhtIntent.setAction(Intent.ACTION_CALL); //这个快捷方式用来打电话
			dowhtIntent.setData(Uri.parse("tel://110")); //电话号码
			intent.putExtra(Intent.EXTRA_SHORTCUT_NAME, "哈哈");  //快捷方式的名字
			intent.putExtra(Intent.EXTRA_SHORTCUT_ICON, BitmapFactory.decodeResource(getResources(), R.drawable.ic_launcher));		//快捷方式的图标
	
			intent.putExtra(Intent.EXTRA_SHORTCUT_INTENT, dowhtIntent);//装载数据
			sendBroadcast(intent);
		}
	}

	/*
	 *创建我们的App的快捷方式
	 */
	public class StartActivity extends Activity {
	
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
	
		}
	
		private void createShortcut() {
		
			Intent intent = new Intent();	
			intent.setAction("com.android.launcher.action.INSTALL_SHORTCUT");
	
			//如果设置为true表示可以创建重复的快捷方式
			intent.putExtra("duplicate", false);
			intent.putExtra(Intent.EXTRA_SHORTCUT_ICON, BitmapFactory.decodeResource(getResources(), R.drawable.ic_launcher));
			intent.putExtra(Intent.EXTRA_SHORTCUT_NAME, "我的App");
		
			/**
			 *说明，我们这个快捷方式要启动应用
			 * 这个地方不能使用显示意图
			 * 必须使用隐式意图来启动我们的应用
			 */
			Intent shortcut_intent = new Intent();	
			shortcut_intent.setAction("start_my_app");	 //我们应用的首页的<intent-filter/>, 具体配置在下面
			shortcut_intent.addCategory("android.intent.category.DEFAULT");  //得加上
		
			intent.putExtra(Intent.EXTRA_SHORTCUT_INTENT, shortcut_intent);		
			sendBroadcast(intent);
			
		}
	}
	
	
 	 <activity android:name="com.itheima.mobileguard.activities.HomeActivity" >
            <intent-filter>
                <!-- 这个名字可以随便取 -->
                <action android:name=start_my_app" ></action>
                <category android:name="android.intent.category.DEFAULT" ></category>
            </intent-filter>
     </activity>

