title: Android项目-多个点击事件走一个onClick方法
date: 2016/3/4 18:26:29     
categories: Android
---

# Android项目-多个点击事件走一个onClick方法 #

- 在Android中给一个控件设置点击事件的写法有很多种
	- 1.在Activity中写一个内部类实现onClickListener接口，设置事件时，直接这个内部类的实例
		- View.setOnClickListener(new myInnerListener)
	- 2.直接使用匿名内部类， 原理和上面一样
	- 3.在XML文件中配置onClick属性，在Activity实现对应的方法
	- 4.让Activity实现onClickListener接口，然后复写onClick方法，然后对点击事件进行统一处理

- 下面来看一下第4种写法


public class ClickActivity extends Activity implements OnClickListener {
	

	private Button btnOne;
	private Button btnTwo;
	private Button btnThree;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_app_manager);
		btnOne = (TextView) findViewById(R.id.btnOne);
		btnTwo = (TextView) findViewById(R.id.btnTwo);
		btnThree = (ListView) findViewById(R.id.btnThree);

		btnOne.setOnCLickListener(this);
		btnTwo.setOnCLickListener(this);
		btnThree.setOnCLickListener(this);

	}

	@Override
	public void onClick(View v) {
		switch (v.getId()) {
			case R.id.btnOne:
				
				break;
			case R.id.btnTwo:
				
				break;
			case R.id.btnThree:
				
				break;
		}
	}
}