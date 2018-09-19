title: Android进阶-Adminastration与手机定位
date: 2016/3/1 19:12:54     
categories: Android
---

# Android进阶-Adminastration与手机定位#

## Adminastration ##

> 我们可以使用它来管理、控制我们的Android设备。在Android系统上，我们可以查看所拥有的设备管理器， 一般在手机的安全设置选项中。

- 那么具体怎么使用呢？（怎么创建一个设备管理器应用）
- 我们可以使用Adminastration的API去编写一个管理应用安装在设备， 然后这个应用就可以对设备进行管理。
- 比如：锁屏、清除数据、设置开机密码等

> NT：若想使用设备管理器应用，必须先激活设备管理器。

下面是官方文档给出的使用范例：

- 定义一个 DeviceAdminReceiver的子类，（定义在这里就可以， 不需要做别的）， 这个receiver在清单文件中需要注册以下内容
	- 获取 BIND_DEVICE_ADMIN permission.
	- intent filter中设置 ACTION_DEVICE_ADMIN_ENABLED intent，使其能够接收这个广播
	- A declaration of security policies used in metadata.

以上3条， 体现在清单文件中是这样的：

	<!-- 你的应用-->
	<activity android:name=".app.DeviceAdminSample"
	            android:label="@string/activity_sample_device_admin">
	    <intent-filter>
	        <action android:name="android.intent.action.MAIN" />
	        <category android:name="android.intent.category.SAMPLE_CODE" />
	    </intent-filter>
	</activity>
	
	<!--必须定义的DeviceAdminReceiver的子类-->
	<receiver android:name=".app.DeviceAdminSample$DeviceAdminSampleReceiver"
	        android:label="@string/sample_device_admin"      
	        android:description="@string/sample_device_admin_description"
	        android:permission="android.permission.BIND_DEVICE_ADMIN">
	    <meta-data android:name="android.app.device_admin"
	            android:resource="@xml/device_admin_sample" />
	    <intent-filter>
	        <action android:name="android.app.action.DEVICE_ADMIN_ENABLED" />
	    </intent-filter>
	</receiver>


- 对于xml/device_admin_sample   在这个XML文件中，我们应定义我们这个设备管理器应用说拥有的能力（即能干什么），内容：

----------
	
	<device-admin xmlns:android="http://schemas.android.com/apk/res/android">
	  <uses-policies>
	    <limit-password />
	    <watch-login />
	    <reset-password />
	    <force-lock />
	    <wipe-data />
	    <expire-password />
	    <encrypted-storage />
	    <disable-camera />>
	  </uses-policies>
	</device-admin>


- 当你完成了DeviceAdminReceiver的子类的上面的所用工作， 你就可以在你的代码中，使用Adminastration的API去管理设备了。
- 但是在使用这个设备管理器（应用）之前， 你应该激活它， 可以再手机的设置->安全->设备管理器中手动激活， 也可以代码激活。

下面是一个简单的设备管理器应用， 它实现了一键锁屏：

	public class MainActivity extends Activity {
	
		private DevicePolicyManager mDPM;
		private ComponentName mDeviceAdminSample;
	
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
	
			mDPM = (DevicePolicyManager) getSystemService(Context.DEVICE_POLICY_SERVICE);// 获取设备策略服务
			mDeviceAdminSample = new ComponentName(this, AdminReceiver.class);// 设备管理组件
	
			mDPM.lockNow();// 立即锁屏     （在mDeviceAdminSample应用被激活的情况下）
			finish();
		}
	
		// 代码激活设备管理器, 也可以在设置->安全->设备管理器中手动激活
		public void activeAdmin(View view) {
			Intent intent = new Intent(DevicePolicyManager.ACTION_ADD_DEVICE_ADMIN);
			intent.putExtra(DevicePolicyManager.EXTRA_DEVICE_ADMIN,
					mDeviceAdminSample);
			intent.putExtra(DevicePolicyManager.EXTRA_ADD_EXPLANATION,
					"一键锁屏");
			startActivity(intent);
		}
	
		//代码卸载
		public void unInstall(View view) {

			mDPM.removeActiveAdmin(mDeviceAdminSample);// 取消激活

			// 卸载程序
			Intent intent = new Intent(Intent.ACTION_VIEW);
			intent.addCategory(Intent.CATEGORY_DEFAULT);
			intent.setData(Uri.parse("package:" + getPackageName()));  //要卸载应用的包名
			startActivity(intent);
		}
	}	


其他功能还有很多例如： 
    
	DevicePolicyManager.wipeData(0);// 清除数据,恢复出厂设置
	DevicePolicyManager.resetPassword("123456", 0);  //重置开机密码

**注意：这个一键锁屏应用在激活的情况下是卸载不下的， 必须先把他Disable了， 上面代码也实现的代码卸载**


## 手机定位 ##
很多方式都可以实现手机定位：

- 网路定位，即依赖IP地址定位（缺点是动态IP不准确， 范围太广）
- 基站定位（通讯公司的信号塔那种东西，缺点也是范围太广， 而且具有地域性）
- GPS定位（卫星定位， 比较精确， 但是易受干扰）
- A-GPS定位（辅助GPS定位）
	- 通过网络和GPS共同定位， 一般Android手机都采用这个方式


那么我们如何使用Android提供的定位功能呢？

1） 首先 ， 要想在代码中使用定位相关API，应获取下面3个权限：

    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_MOCK_LOCATION"/>

2） 主要依靠LocationManager

步骤：

	- 获取系统的LOCATION_SERVICE服务（即主要依靠LocationManager）
	- 选择你所采用的定位方式Provider
	- 实现LocationListener接口

代码范例：

	public class MainActivity extends Activity {

	private TextView tvLocation;
	private LocationManager lm;
	private MyLocationListener listener;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		tvLocation = (TextView) findViewById(R.id.tv_location);

		lm = (LocationManager) getSystemService(LOCATION_SERVICE);
		//List<String> allProviders = lm.getAllProviders();// 获取所有位置提供者
		//System.out.println(allProviders);
		listener = new MyLocationListener();
		lm.requestLocationUpdates(LocationManager.GPS_PROVIDER, 0, 0, listener);// 参1表示位置提供者,参2表示最短更新时间,参3表示最短更新距离
	}

	class MyLocationListener implements LocationListener {

		// 位置发生变化
		@Override
		public void onLocationChanged(Location location) {
			String j = "经度:" + location.getLongitude();
			String w = "纬度:" + location.getLatitude();
			String accuracy = "精确度:" + location.getAccuracy();
			String altitude = "海拔:" + location.getAltitude();

			/*do something*/
		}

		// 位置提供者状态发生变化
		@Override
		public void onStatusChanged(String provider, int status, Bundle extras) {
		}

		// 用户打开gps
		@Override
		public void onProviderEnabled(String provider) {
		}

		// 用户关闭gps
		@Override
		public void onProviderDisabled(String provider) {
		}

	}
	}

## 零碎 ##

- Point 1

>  如果把媒体文件，放在项目中，则应该放在res/raw目录下

- Point2（获取短信， 短信拦截）

1. 首先，肯定要定义广播接收者，来接收系统短信广播
	- 如果我们想让我们的广播接收者的优先级最高， 可以 将 android:priority="2147483647 （即int的最大值），
2. 获取短信内容， 拦截短信

----------
	
	public class SmsReceiver extends BroadcastReceiver {

	@Override
	public void onReceive(Context context, Intent intent) {
		Object[] objects = (Object[]) intent.getExtras().get("pdus");

		for (Object object : objects) {// 短信最多140字节,
										// 超出的话,会分为多条短信发送,所以是一个数组,因为我们的短信指令很短,所以for循环只执行一次
			SmsMessage message = SmsMessage.createFromPdu((byte[]) object);
			String originatingAddress = message.getOriginatingAddress();// 短信来源号码
			String messageBody = message.getMessageBody();// 短信内容

			/* do something*/

			abortBroadcast();
	
		}
	}

	}



