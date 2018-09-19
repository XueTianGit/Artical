title: Android-Boradcast
date: 2016/2/28 10:31:15 
categories: Android
---

# 广播 #
广播（Broadcast）作为Android四大组件之一，重要性不言而喻。

## 什么是Android广播 ##
> 系统运行时，会产生很多事件（比如：电量改变， 收发短信， 拨打电话， 屏幕解锁），那么某些事件产生时，系统就会发送一个广播来告诉应用我怎么怎么了。那么，应用就可以根据广播来做出相应的反应。

### 广播的分类 ###

* 普通广播：所有跟广播的intent匹配的广播接收者都可以收到该广播，并且是没有先后顺序（同时收到）,消息传递的效率比较高。
	缺点：接收者不能将处理结果传递给下一个接收者，并且无法终止广播Intent的传播；
	如何发送： Context.sendBroadcast()
* 有序广播：所有跟广播的intent匹配的广播接收者都可以收到该广播，但是会按照广播接收者的优先级来决定接收的先后顺序##  ##
	* 优先级的定义：-1000~1000
	* 最终接收者：所有广播接收者都接收到广播之后，它才接收，并且一定会接收
	* abortBroadCast：阻止其他接收者接收这条广播，类似拦截，只有有序广播可以被拦截
	缺点：消息传递的效率比较低
	如何发送：Context.sendOrderedBroadcast()
NT：有序广播，系统会根据接收者声明的优先级别按顺序逐个执行接收者，前面的接收者有权终止广播**(BroadcastReceiver.abortBroadcast())**，如果广播被前面的接收者终止，后面的接收者就再也无法获取到广播。对于有序广播，前面的接收者可以将处理结果存放进广播Intent，然后传给下一个接收者。

## 广播接收者 ##
> 系统产生广播，应用通过使用广播接收者（BroadcastReceiver）就可以来收听广播， 进而产生相应的行为。

### 创建广播接收者 ###
1. 定义java类继承BroadcastReceiver，并且重写onReceive()方法。
2. 在清单文件中定义receiver节点，定义name属性，指定广播接收者java类的全类名
3. 在intent-filter的节点中，指定action子节点，action的值必须跟要接收的广播中的action匹配，比如，如果要接收打电话广播，那么action的值必须指定为：
<action android:name="android.intent.action.NEW_OUTGOING_CALL"/>
NT：因为打电话广播中所包含的action，就是"android.intent.action.NEW_OUTGOING_CALL"，所以我们定义广播接收者时，action必须与其匹配，才能收到这条广播
即便广播接收者所在进程已经被关闭，当系统发出的广播中的action跟该广播接收者的action匹配时，**系统会启动该广播接收者所在的进程，并把广播发给该广播接收者**
**并且：应注意收听广播的权限问题**
NT：通过在清单文件中配置接收的广播，叫做静态订阅广播



> 对于第2、3步，我们还可以在代码中实现，这叫动态订阅广播
> IntentFilter filter = newIntentFilter("android.provider.Telephony.SMS_RECEIVED");  
> IncomingSMSReceiver receiver = newIncomingSMSReceiver();  
> registerReceiver(receiver, filter); 


## 自定义发送广播 ##
1. 发送广播即发送一个intent sendBoradcast(intent);
2. 发送时应设置action， intent.setAction("com.suixin.mybroadcast");
example:

    	Intent intent = new Intent();
    	//广播中的action也是自定义的
    	intent.setAction("com.itheima.zdy");
    	sendBroadcast(intent);

NT:
接收自定义广播时，按正常接收就OK了， 只不过在配置时应注意：

		<receiver android:name="com.itheima.receivezdy.ZDYReceiver">
            <intent-filter >
                <action android:name="com.suixin.mybroadcast"/>
            </intent-filter>
        </receiver>
## 广播应用的举例 ##

### IP拨号器 
> 这个应用可以在你打电话时，自动帮你在你所拨打的电话号码前面加上指定号码。
> 原理：接收拨打电话的广播，修改广播内携带的电话号码

1） 定义广播接收者接收打电话广播

	public class CallReceiver extends BroadcastReceiver {

		//当广播接收者接收到广播时，此方法会调用
		@Override
		public void onReceive(Context context, Intent intent) {
			//拿到用户拨打的号码
			String number = getResultData();
			//修改广播内的号码
			setResultData("17951" + number);
		}
	}

2）在清单文件中定义该广播接收者接收的广播类型

		<receiver android:name="com.itheima.ipdialer.CallReceiver">
            <intent-filter >
                <action android:name="android.intent.action.NEW_OUTGOING_CALL"/>
            </intent-filter>
        </receiver>

3）接收打电话广播需要权限

		<uses-permission android:name="android.permission.PROCESS_OUTGOING_CALLS"/>

NT： 即使广播接收者的进程没有启动，当系统发送的广播可以被该接收者接收时，系统会自动启动该接收者所在的进程

###短信拦截器
>系统收到短信时会产生一条广播，广播中包含了短信的号码和内容

* 定义广播接收者接收短信广播

		public void onReceive(Context context, Intent intent) {
		//拿到广播里携带的短信内容
		Bundle bundle = intent.getExtras();
		Object[] objects = (Object[]) bundle.get("pdus");
		for(Object ob : objects ){
			//通过object对象创建一个短信对象
			SmsMessage sms = SmsMessage.createFromPdu((byte[])ob);
			System.out.println(sms.getMessageBody());
			System.out.println(sms.getOriginatingAddress());
		}
	}
* 系统创建广播时，把短信存放到一个数组，然后把数据以pdus为key存入bundle，再把bundle存入intent
* 清单文件中配置广播接收者接收的广播类型，注意要设置优先级属性，要保证优先级高于短信应用，才可以实现拦截

		<receiver android:name="com.itheima.smslistener.SmsReceiver">
            <intent-filter android:priority="1000">
                <action android:name="android.provider.Telephony.SMS_RECEIVED"/>
            </intent-filter>
        </receiver>
* 添加权限

		<uses-permission android:name="android.permission.RECEIVE_SMS"/>

* 4.0以后广播接收者安装以后必须手动启动一次，否则不生效
* 4.0以后广播接收者如果被手动关闭，就不会再启动了

---
###监听SD卡状态
* 清单文件中定义广播接收者接收的类型，监听SD卡常见的三种状态，所以广播接收者需要接收三种广播

		 <receiver android:name="com.itheima.sdcradlistener.SDCardReceiver">
            <intent-filter >
                <action android:name="android.intent.action.MEDIA_MOUNTED"/>
                <action android:name="android.intent.action.MEDIA_UNMOUNTED"/>
                <action android:name="android.intent.action.MEDIA_REMOVED"/>
                <data android:scheme="file"/>
            </intent-filter>
        </receiver>
* 广播接收者的定义

		public class SDCardReceiver extends BroadcastReceiver {
			@Override
			public void onReceive(Context context, Intent intent) {
				// 区分接收到的是哪个广播
				String action = intent.getAction();
					
				if(action.equals("android.intent.action.MEDIA_MOUNTED")){
					System.out.println("sd卡就绪");
				}
				else if(action.equals("android.intent.action.MEDIA_UNMOUNTED")){
					System.out.println("sd卡被移除");
				}
				else if(action.equals("android.intent.action.MEDIA_REMOVED")){
					System.out.println("sd卡被拔出");
				}
			}
		}

---
###勒索软件---开机广播
* 接收开机广播，在广播接收者中启动勒索的Activity
* 清单文件中配置接收开机广播

		<receiver android:name="com.itheima.lesuo.BootReceiver">
            <intent-filter >
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
            </intent-filter>
        </receiver>
* 权限

		<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>

* 定义广播接收者

		@Override
		public void onReceive(Context context, Intent intent) {
			//开机的时候就启动勒索软件
			Intent it = new Intent(context, MainActivity.class);		
			context.startActivity(it);
		}
* **以上代码还不能启动MainActivity，因为广播接收者的启动，并不会创建任务栈，那么没有任务栈，就无法启动activity**
* 手动设置创建新任务栈的flag

		it.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);

---
###监听应用的安装、卸载、更新
> 原理：应用在安装卸载更新时，系统会发送广播，广播里会携带应用的包名
* 清单文件定义广播接收者接收的类型，因为要监听应用的三个动作，所以需要接收三种广播

		<receiver android:name="com.itheima.app.AppReceiver">
            <intent-filter >
                <action android:name="android.intent.action.PACKAGE_ADDED"/>
                <action android:name="android.intent.action.PACKAGE_REPLACED"/>
                <action android:name="android.intent.action.PACKAGE_REMOVED"/>
                <data android:scheme="package"/>
            </intent-filter>
        </receiver>
* 广播接收者的定义

		public void onReceive(Context context, Intent intent) {
			//区分接收到的是哪种广播
			String action = intent.getAction();
			//获取广播中包含的应用包名
			Uri uri = intent.getData();
			if(action.equals("android.intent.action.PACKAGE_ADDED")){
				System.out.println(uri + "被安装了");
			}
			else if(action.equals("android.intent.action.PACKAGE_REPLACED")){
				System.out.println(uri + "被更新了");
			}
			else if(action.equals("android.intent.action.PACKAGE_REMOVED")){
				System.out.println(uri + "被卸载了");
			}
		}






