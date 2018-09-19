title: Andorid-基础复习
date: 7/22/2016 8:56:49 
categories: Android
---



#Activity中调用Service中的方法

- 最直接的就是，直接用  ServiceConnection 的类的onServiceConnected（）回调方法中获取到 Service 的 onBind（）方法 
  return 过来的 Binder 的子类。
- 再者就是使用, handler。
	- 在绑定Service时， Service返回其handler。
	- Activity获的这个handler，利用他与Service进行通信
- 对用不同应用之间， 可以使用AIDL

##Service与Activity中互相通信
- 方法是，拿到各自的handle	r	
	- Activity的handler可以在"绑定"时发送给 Service
	- Service在handlerMessage方法中，根据不同的消息
		- 或获得Activtiy的handler
		- 或处理handler发送的消息
	- Service可以通过Activity的handler与Activity通信

>代码：
	
	 class MyServiceConnection implements ServiceConnection {
		
		 @Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			messenger = new Messenger(service);  //这个 service  其实是Service的handler对象
			Message message=new Message();
			 message.what = 1;
			//Activity 绑定 Service 的时候给 Service 发送一个消息，该消息的 obj 属性是一个 Messenger 对象
			 message.obj = mOutMessenger;
			try {
			 messenger.send(message);
			} catch (RemoteException e) {
			e.printStackTrace();
			}
		}

	public class MessengerService extends Service {

		private Messenger messenger = new Messenger(new IncomingHandler());   //传入handler
		private Messenger mActivityMessenger ;
		@Override
		public IBinder onBind(Intent intent) {
			IBinder binder = messenger.getBinder();
			return binder;
		}
	/*.....*/
	}


#Android中布局优化的措施

- 尽可能减少布局的嵌套层级
	- 可以使用 sdk 提供的 hierarchyviewer 工具分析视图树，帮助我们发现没有用到的布局。
- 不用设置不必要的背景，避免过度绘制
	- 比如父控件设置了背景色，子控件完全将父控件给覆盖的情况下，那么父控件就没有必要设置背景。
- 使用<include>标签复用相同的布局代码
- 使用<merge>标签减少视图层次结构， 该标签主要有两种用法：
	-  因为所有的 Activity视图的根节点都是 FrameLayout ，因此如果我们的自定义的布局也是FragmenLayout的时候那么可以使用 merge 替换。
	-  当应用 Include 或者 ViewStub 标签从外部导入 xml 结构时，可以将被导入的 xml 用 merge 作为根节点表示，
	   这样当被嵌入父级结构中后可以很好的将它所包含的子集融合到父级结构中，而不会出现冗余的节点。
	- NOTE：<merge>只能作为 xml 布局的根元素。
- 通过<ViewStub>实现 View 的延迟加载
>代码：

		<ViewStub
		android:id="@+id/vs"
		android:layout_width="match_parent"
		android:layout_height="0dp"
		android:layout_weight="1"
		android:inflatedId="@+id/my_view"
		android:layout="@layout/my_layout" />

		public void loadVS(View view){
			ViewStub vs = (ViewStub) findViewById(R.id.vs);
			View inflate = vs.inflate();
			int inflatedId = vs.getInflatedId();
			int id = inflate.getId();
			Toast.makeText(this, "inflatedId="+inflatedId+"***"+"id="+id,
			Toast.LENGTH_SHORT).show();
		}

#ListView如何提高效率
- 复用 ConvertView
- 自定义静态类 ViewHolder
	- 非静态内部类拥有外部类对象的强引用，因此为了避免对外部类（外部类很可能是 Activity）对象的引用，那么最好将内部类声明为 static 的。
- 使用分页加载
- 使用 WeakRefrence 引用 ImageView 对象

#在ScrollView中嵌套ListView
> 在 ScrollView 添加一个 ListView 会导致 listview 控件显示不全，通常只会显示一条，这是因为两个控件的滚动
事件冲突导致。所以需要通过 listview 中的 item 数量去计算 listview 的显示高度，从而使其完整展示


#在 Android 中如何调用 C 语言
> 当我们的 Java 需要调用 C 语言的时候可以通过 JNI 的方式，Java Native Interface。Android 提供了对 JNI 的支
持， 因此我们在Android中可以使用 JNI调用C 语言。 在Android开发目录的 libs 目录下添加 xxx.so 文件， 不过 xxx.so
文件需要放在对应的 CPU 架构名目录下，比如 armeabi，x86 等。
在 Java 代码需要通过 System.loadLibrary(libName); 加载 so 文件。同时 C 语言中的方法在 java 中必须
以 native 关键字来声明。普通 Java 方法调用这个 native 方法接口，虚拟机内部自动调用 so 文件中对应的方法。

#Fragment的replace和add方法的区别
> Fragment 的容器一个 FrameLayout，add 的时候是把所有的 Fragment 一层一层的叠加到了 FrameLayout 上
了，而 replace 的话首先将该容器中的其他 Fragment 去除掉然后将当前 Fragment 添加到容器中。
一个 Fragment 容器中只能添加一个 Fragment 种类，如果多次添加则会报异常，导致程序终止，而 replace 则
无所谓，随便切换。
**因为通过 add 的方法添加的 Fragment，每个 Fragment 只能添加一次，因此如果要想达到切换效果需要通过
Fragment 的的 hide 和 show 方法结合者使用。将要显示的 show 出来，将其他 hide 起来。这个过程 Fragment 的
生命周期没有变化。**
通 过 replace 切 换 Fragment ， 每 次 都 会 执 行 上 一 个 Fragment 的 onDestroyView ， 新 Fragment 的
onCreateView、onStart、onResume 方法。


#什么情况下会导致内存泄露
- 资源释放问题
	- 程序代码的问题，长期保持某些资源，如 Context、Cursor、IO 流的引用，资源得不到释放造成内存泄露。
- 对象内存过大问题
	- 保存了多个耗用内存过大的对象（如 Bitmap、XML 文件），造成内存超出限制。
- static 关键字的使用问题
	- atic 是 Java 中的一个关键字，当用它来修饰成员变量时，那么该变量就属于该类，而不是该类的实例。所
	以用 static 修饰的变量，它的生命周期是很长的，如果用它来引用一些资源耗费过多的实例（Context 的情况最
	多），这时就要谨慎对待了
	- 解决方案
		- 应该尽量避免 static 成员变量引用资源耗费过多的实例，比如 Context。
		- Context 尽量使用 ApplicationContext，因为 Application 的 Context 的生命周期比较长，引用它不会
			出现内存泄露的问题。
		- 使用 WeakReference 代替强引用。比如可以使用 WeakReference<Context> mContextRef;
- 线程导致内存溢出
	- 线程产生内存泄露的主要原因在于线程生命周期的不可控(AsyncTask更不可控)。解决方案如下：
		- 将线程的内部类，改为静态内部类（因为非静态内部类拥有外部类对象的强引用，而静态类则不拥有） 。
		- 在线程内部采用弱引用保存 Context 引用。

#如何避免 OOM 异常
- 图片过大导致 OOM
	- 等比例缩小图片
	- 对图片采用软引用，及时地进行 recyle()操作
- 界面切换导致 OOM (横竖屏切换 N 次后 OOM 了)
- 查询数据库没有关闭游标
- 构造 Adapter 时，没有使用缓存的 convertView
- Bitmap 对象不再使用时调用 recycle()释放内存.0

#Android应用性能优化
[http://blog.csdn.net/yanbober/article/details/48394201](http://blog.csdn.net/yanbober/article/details/48394201)


#屏幕适配
- 对于 drawable、dimens、layout文件夹，我们都了可以创建对应手机尺寸的文件夹来达到屏幕适配的效果
- 权重适配
	- 在控件中使用属性 android:layout_weight="1" 可 以起到适配效果，但是该属性的使用有如下规则：
		- 只能用在线性控件中，比如 LinearLayout。
		- 竖直方向上使用权重的控件高度必须为 0dp（Google 官方的推荐用法）
		- 水平方向上使用权重的控件宽度必须为 0dp（Google 官方的推荐用法） 
- 多使用RelativeLayout

#View的绘制流程
> 整个 View 树的绘图流程是在 ViewRoot.java 类(该类位于 Android 源码下面：
D:\AndroidSource_GB\AndroidSource_GB\frameworks\base\core\java\android\view)的
performTraversals()函数展开的，该函数做的执行过程可简单概况为根据之前设置的状态，判断是否需要重新计
算视图大小(measure)、是否重新需要安置视图的位置(layout)、以及是否需要重绘 (draw)

- mesarue()过程
	- 主要作用：为整个 View 树计算实际的大小，即设置实际的高(对应属性:mMeasuredHeight)和宽(对应属性:mMeasureWidth)，
	  每个 View 的控件的实际宽高都是由父视图和本身视图决定的
	- 具体的调用链如下： ViewRoot 根对象的属性 mView(其类型一般为 ViewGroup 类型)调用 measure()方法去
		计算 View 树的大小，回调 View/ViewGroup 对象的 onMeasure()方法， 该方法实现的功能如下：
		- 设置本 View 视图的最终大小，该功能的实现通过调用 setMeasuredDimension()方法去设置实际
			的高(对应属性：mMeasuredHeight)和宽(对应属性：mMeasureWidth)。
		- 如果该 View 对象是个 ViewGroup 类型，需要重写该 onMeasure()方法，对其子视图进行遍历的
			measure() 过 程 。 对 每 个 子 视 图 的 measure() 过 程 ， 是 通 过 调 用 父 类 ViewGroup.java 类 里 的
			measureChildWithMargins()方法去实现，该方法内部只是简单地调用了 View 对象的 measure()方法
- layout 布局过程
	- 主要作用：为将整个根据子视图的大小以及布局参数将 View 树放到合适的位置上。
	- layout 方法会设置该 View 视图位于父视图的坐标轴，即 mLeft，mTop，mLeft，mBottom(调用
	setFrame()函数去实现)接下来回调 onLayout()方法(如果该 View 是 ViewGroup 对象，需要实现该方法，对每个子视
	图进行布局)。
	- 如果该 View 是个 ViewGroup 类型，需要遍历每个子视图 chiildView，调用该子视图的 layout()方法去设置它的坐标值。
- draw()绘图过程
	- 由 ViewRoot 对象的 performTraversals()方法调用 draw()方法发起绘制该 View 树， 值得注意的是每次发起绘
	图时，并不会重新绘制每个 View 树的视图，而只会重新绘制那些“需要重绘”的视图，View 类内部变量包含了一个
	标志位 DRAWN，当该视图需要重绘时，就会为该 View 添加该标志位。
	- 调用流程 ：
		- 绘制该 View 的背景
		- 为显示渐变框做一些准备操作(大多数情况下，不需要改渐变框)
		- 调用 onDraw()方法绘制视图本身(每个 View 都需要重载该方法，ViewGroup 不需要实现该方法)
		- 调用 dispatchDraw ()方法绘制子视图(如果该 View 类型不为 ViewGroup，即不包含子视图，不需要重载该方法)
			值得说明的是， ViewGroup 类已经为我们重写了 dispatchDraw ()的功能实现， 应用程序一般不需要重写该方法，
			但可以重载父类函数实现具体的功能。