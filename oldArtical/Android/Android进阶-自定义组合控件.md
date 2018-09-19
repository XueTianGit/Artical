title: Android项目-自定义组合控件
date: 2016/3/7 18:27:27      
categories: Android
---

# Android项目-自定义组合控件#

- Point1 (禁掉一个控件的点击事件)
	
以CheckBox为例：

    <CheckBox
        android:clickable="false"
        android:focusable="false"
        android:focusableInTouchMode="false" />

- Point2 (使用Handler来对主线程进行延迟)

例如：

	handler.sendEmptyMessageDelayed(EMPTY_MESSAGE, 2000);// 延时2秒后发送消息	

- Point3 （自定义组合控件）
	-  一般步骤
		1. 自定义一个View, 继承ViewGroup, 比如RelativeLayout
		2. 编写组合控件的布局文件,在自定义的View中加载（即每个构造方法都要加载）
				// 将自定义好的布局文件设置给当前的自定义控件
				View.inflate(getContext(), R.layout.view_setting_item, this);
	
	- 给自定义的组合控件添加属性
		1. 在values/arrts.xml中创建我们的控件所需要的属性
		2. 在我么的自定义组合控件中，解析这些属性， 并提供给使用者设置属性的接口

范例代码：
	
	//自定义组合控件的View， 该View中有两个TextView， 和一个CheckBox
	public class SettingItemView extends RelativeLayout {
	
		private static final String NAMESPACE = "http://schemas.android.com/apk/res/com.suixin.mobileguard";
		private TextView tvTitle;      //所含控件的成员变量
		private TextView tvContent;    //所含控件的成员变量
		private CheckBox cb;           //所含控件的成员变量
		private String mTitle;  //使用者传来的属性值
		private String mDescOn; //使用者传来的属性值
		private String mDescOff; //使用者传来的属性值
	
		public SettingItemView(Context context, AttributeSet attrs, int defStyleAttr) {
			super(context, attrs, defStyleAttr);
			init();
		}
	
		//如果使用者， 在布局文件中使用属性的话， 在实例化SettingItemView对象时，
		//这个构造方法就会被调用
		public SettingItemView(Context context, AttributeSet attrs) {
			super(context, attrs);
			mTitle = attrs.getAttributeValue(NAMESPACE, "title");// 根据属性名称,获取属性的值
			mDescOn = attrs.getAttributeValue(NAMESPACE, "desc_on");
			mDescOff = attrs.getAttributeValue(NAMESPACE, "desc_off");
			init();
		}
	
		public SettingItemView(Context context) {
			super(context);
			init();
		}
		
		private void init(){
			View settingItemView  = View.inflate(getContext(), R.layout.setting_item, this);
			tvTitle = (TextView) settingItemView.findViewById(R.id.tv_title);
			tvContent = (TextView) settingItemView.findViewById(R.id.tv_content);
			cb = (CheckBox) settingItemView.findViewById(R.id.cb_status);
		}
		
		public void setTitle(String mTitle) {
			tvTitle.setText(mTitle);
		}
		
		public void setDesc(String desc){
			tvContent.setText(desc);
		}
		
		public boolean isChecked(){
			return cb.isChecked();
		}
		
		
		public void setChecked(boolean check) {
			cb.setChecked(check);
	
			// 根据选择的状态,更新文本描述
			if (check) {
				setDesc(mDescOn);
			} else {
				setDesc(mDescOff);
			}
		}
	
	}
	
	// values/attrs.xml文件中的内容
	    <declare-styleable name="SettingItemView">
	        <attr name="title" format="string" />
	        <attr name="desc_on" format="string" />
	        <attr name="desc_off" format="string" />
   		</declare-styleable>
	//使用范例
		 <com.suixin.mobileguard.view.SettingItemView
	        android:id="@+id/siv_updateItem"
	        suixin:title="自动更新设置"
	        suixin:desc_on="自动更新开启"
	        suixin:desc_off="自动更新关闭"      
	        />


	- 小总结
		- Android中的每个控件都有对应的View对象
		- 控件的属性，应使用attrs.xml文件描述
		- 描述的属性，我们对应的View对象，就要对其负责解析，还可以暴露代码设置接口
		- 每个控件的View对象都是针对XML来设计的



		




