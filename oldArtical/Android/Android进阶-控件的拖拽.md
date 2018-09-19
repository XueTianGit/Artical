title: Android进阶-控件的拖拽
date: 2016/3/1 19:13:57    
categories: Android
---

# Android进阶-控件的拖拽 #

> 需求：使控件可以再屏幕上自由拖拽。

思路：

- 可以在 View.setOnTouchListener()中监听控件的触摸事件，在触摸事件中我们应做以下处理
	- 记下控件的起始坐标    (ACTION_DOWN)
	- 计算控件的移动偏移量  （ACTION_MOVE）
	- 更新控件的位置		  (ACTION_MOVE）	
		- 使用View.layout()
	- 重新初始化控件起始坐标，以便下次更新


示例代码要点：

1) View.layout()并不可以在Activity的onCreate()方法中调用

> 这是因为Android系统在绘制界面时要经过3个过程： onMeasure(), onLayout(), onDraw()
> 可以联想到View.layout()是在onLayout()时调用的. Activity的onCreate()方法可定是在这三个方法之前调用的
> 如果你在onCrate()方法中就给控件设置了布局参数， 到后面这三个方法调用时也会被覆盖吧（我猜的）
> 所以View.layout()不能再onCreate()方法中调用

2) 那么如何在onCreate()方法中设置控件的布局位置呢？(就类似于代码布局了)
> 其实很简单，我们只需在android系统绘制界面之前，改一下布局参数就可以了，
> 即调用View.setLayoutParams(layoutParams);// 重新设置位置, 

3)上面两点与下面代码并没什么关系，下面代码应注意

	1. ivDrag.layout(l, t, r, b); 的4个参数， 分别代表 
		> l：屏幕左边界到控件左边界的距离
		> t：屏幕上边界到控件上边界的距离
		> r：屏幕左边界到控件右边界的距离
		> b：屏幕上边界到控件下边界的距离
	2. 在使用ivDrag.layout(l, t, r, b);动态更新控件位置时，不要忘记判断与屏幕边界的问题
		>例如：控件右边界不能超出屏幕右边界。如果超出， 控件的显示可能会和实际有差别
	3. 控件位置更新完之后，应重新初始化，控件的坐标起点
	4. rawX代表在屏幕坐标系下的X坐标

----------

	public class DragViewActivity extends Activity {
	
		private ImageView ivDrag;
	
		private int startX;
		private int startY;
	
	
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_drag_view);
			ivDrag = (ImageView) findViewById(R.id.iv_drag);
	
			// 设置触摸监听
			ivDrag.setOnTouchListener(new OnTouchListener() {
	
				@Override
				public boolean onTouch(View v, MotionEvent event) {
					switch (event.getAction()) {
					case MotionEvent.ACTION_DOWN:
						// 初始化起点坐标
						startX = (int) event.getRawX();
						startY = (int) event.getRawY();
						break;
					case MotionEvent.ACTION_MOVE:
						int endX = (int) event.getRawX();
						int endY = (int) event.getRawY();
	
						// 计算移动偏移量
						int dx = endX - startX;
						int dy = endY - startY;
	
						// 更新左上右下距离
						int l = ivDrag.getLeft() + dx;
						int r = ivDrag.getRight() + dx;
						int t = ivDrag.getTop() + dy;
						int b = ivDrag.getBottom() + dy;
	
						// 判断是否超出屏幕边界, 注意状态栏的高度
						if (l < 0 || r > winWidth || t < 0 || b > winHeight - 20) {
							break;
						}
	
						// 更新界面
						ivDrag.layout(l, t, r, b);
	
						// 重新初始化起点坐标
						startX = (int) event.getRawX();
						startY = (int) event.getRawY();
						break;
					case MotionEvent.ACTION_UP:
						break;
					default:
						break;
					}
					return true;
				}
			});
		}
	}


