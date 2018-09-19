title: Android进阶-屏幕滑动和获取手机联系人
date: 2016/3/1 9:45:11                
categories: Android
---

# Android进阶-屏幕滑动和获取手机联系人 #

## 屏幕滑动 ##

> 如何使Activity之间的切换通过手势滑动来完成呢? 使用onTouchEvent()吗？ 好像是可行？ 但复杂的处理用户动作算法该由你自己来实现了，
> 可爱的Google已经帮我们实现了， 我们可以使用GestureDetetor对象来完成这件事。

实现步骤：

- 将onTouchEvent()委托给GestureDetetor来处理。
- 注册GestureDetetor的GestureListener， 并重写onFling()方法。
	- Fling 可以解释为一抛， 一扔的意思， 即可以理解为响应用户有速度的滑动屏幕事件。


简单的实现屏幕滑动切换Activity的代码，在这个代码中：

- 将左右切换Activity的实现，抽取到BaseActivity来完成
- 若有Activity想要进行切换，只需继承这个BaseActivity，并复写showNextPage 与showPreviousPage方法
- 前提是可以响应 next 与 previous 按钮鼠标点击事件
- 一般有： rawX : 基于整个屏幕的X左边  X:基于控件的X坐标

如下：

	public abstract class BaseActivity extends Activity {

		private GestureDetector mDectector;

		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);				
			mDectector = new GestureDetector(this, new SimpleOnGestureListener() {
		
				/**
				 * 监听手势滑动事件 e1表示滑动的起点,e2表示滑动终点 velocityX表示水平速度 velocityY表示垂直速度
				 */
				@Override
				public boolean onFling(MotionEvent e1, MotionEvent e2,
						float velocityX, float velocityY) {
		
						// 判断纵向滑动幅度是否过大, 过大的话不允许切换界面
						if (Math.abs(e2.getRawY() - e1.getRawY()) > 100) {
							Toast.makeText(MobileGuardBaseActivity.this, "不能这样划哦!",
									Toast.LENGTH_SHORT).show();
							return true;
						}
			
						// 判断滑动是否过慢
						if (Math.abs(velocityX) < 100) {
							Toast.makeText(MobileGuardBaseActivity.this, "滑动的太慢了!",
									Toast.LENGTH_SHORT).show();
							return true;
						}
			
						// 向右划,上一页
						if (e2.getRawX() - e1.getRawX() > 200) {
							showPreviousPage();
							return true;
						}
			
						// 向左划, 下一页
						if (e1.getRawX() - e2.getRawX() > 200) {
							showNextPage();
							return true;
						}
			
						return super.onFling(e1, e2, velocityX, velocityY);
					}
				});
			}
			
			/**
			 * 展示下一页, 子类必须实现
			 */
			public abstract void showNextPage();
			
			/**
			 * 展示上一页, 子类必须实现
			 */
			public abstract void showPreviousPage();
			
			// 点击下一页按钮
			public void next(View view) {
				showNextPage();
			}
			
			// 点击上一页按钮
			public void previous(View view) {
				showPreviousPage();
			}
			
			@Override
			public boolean onTouchEvent(MotionEvent event) {
				mDectector.onTouchEvent(event);// 委托手势识别器处理触摸事件
				return super.onTouchEvent(event);
			}
			
	}

## 获取手机联系人 ##

> 这件事的实现当然是依赖的内容提供者。

1. 为了获取到联系人，共涉及到2张表（可以这么说），他们的内容提供者的Uri为：		
	- content://com.android.contacts/raw_contacts （主要从这个表获取contact_id）
	- content://com.android.contacts/data (实际是view_data视图的URI)

2. 获取代码：
	
		Uri rawContactsUri = Uri.parse("content://com.android.contacts/raw_contacts");
		Uri dataUri = Uri.parse("content://com.android.contacts/data");  //查的不是data表， 查的实际上是view_data视图
		ArrayList<HashMap<String, String>> list = new ArrayList<HashMap<String,String>>();
			
		//raw_contacts表中获取联系人信息的ID
		Cursor queryRawC = getContentResolver().query(rawContactsUri, new String[]{"contact_id"}, null, null, null);
			
		while( queryRawC.moveToNext() ){
			String contactId = queryRawC.getString(0);
			Cursor dataCursor = getContentResolver().query(dataUri, new String[]{"data1", "mimetype"}, "contact_id = ?", new String[]{contactId}, null);
			if (dataCursor != null) {
				HashMap<String, String> map = new HashMap<String, String>();
				while (dataCursor.moveToNext()) {
					String data1 = dataCursor.getString(0);
					String mimetype = dataCursor.getString(1);
					if ("vnd.android.cursor.item/phone_v2".equals(mimetype)) {
						map.put("phone", data1);
					} else if ("vnd.android.cursor.item/name".equals(mimetype)) {
						map.put("name", data1);
					}
				}

				list.add(map);
				dataCursor.close();
			}
		}
			
		queryRawC.close();

应注意：

	-  我们没有去查mimetype相关的表， 而是根据类型直接去判断的
	-  ArrayList<HashMap<String, String>>可以直接用ListView的SimpleAdapter处理

## SIM卡信息 ##
可以使用TelephonyManager来获取， 例如：

	TelephonyManager tm = (TelephonyManager) getSystemService(TELEPHONY_SERVICE);
	String simSerialNumber = tm.getSimSerialNumber();// 获取sim卡序列号

## 零碎 ##

- Point1 
	- 当我们想让用户输入号码时， 可以直接弹出号码输入框，而不是整个键盘： inputType="phone"

- Point2 
	- 当从一个Activity返回时， 结果码有：
		- Activity.RESULT_OK:我们应手动设置这个； Activity.RESULT_CANCEL:当用户直接点解返回键是，返回的就是这个resultCode。
		
