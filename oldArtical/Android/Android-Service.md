title: Android-Service
date: 2016/2/29 8:13:49      
categories: Android
---

# Android-服务 #

## 概述 ##
>- Service（服务）是一个没有用户界面的在后台运行执行耗时操作的应用组件。其他应用组件能够启动Service，并且当用户切换到另外的应用场景，Service将持续在后台运行。
> - 另外，一个组件能够绑定到一个service与之交互（IPC机制），例如，一个service可能会处理网络操作，播放音乐，操作文件I/O或者与内容提供者（content provider）交互，
>- 所有这些活动都是在后台进行。

服务分为:本地服务和远程服务

* 本地服务：指的是服务和启动服务的activity在同一个进程中
* 远程服务：指的是服务和启动服务的activity不在同一个进程中

## 服务的启动 ##
> 服务有两种启动方式：
方式一:

>- 通过startService()启动的服务处于“启动的”状态，一旦启动，service就在后台运行，即使启动它的应用组件已经被销毁了。
>- 通常started状态的service执行单任务并且不返回任何结果给启动者。比如当下载或上传一个文件，当这项操作完成时，**service应该停止它本身。**

方式二：
> 还有一种“绑定”状态的service，通过调用bindService()来启动，一个绑定的service提供一个允许组件与service交互的接口，可以发送请求、获取返回结果，还可以通过夸进程通信来交互（IPC）。
> 绑定的service只有当应用组件绑定后才能运行，多个组件可以绑定一个service，当调用unbind()方法时，这个service就会被销毁了。并且,当启动服务的进程挂掉时，这个“绑定的”服务也会挂掉。

-> startService()启动 
        intent = new Intent(this, MyService.class);
    	startService(intent);

    	stopService(intent);  //关闭服务

-> bindService()启动

        intent = new Intent(this, MyService.class);
        conn = new MyServiceConn();    
    	bindService(intent, conn, BIND_AUTO_CREATE);

    	unbindService(conn); //解绑服务 

		class MyServiceConn implements ServiceConnection{

	    	//服务连接成功时，此方法调用
			@Override
			public void onServiceConnected(ComponentName name, IBinder service) {
				// TODO Auto-generated method stub
				
			}
	
			//服务失去连接时，此方法调用
			@Override
			public void onServiceDisconnected(ComponentName name) {
				// TODO Auto-generated method stub
				
			}
    	
 	   }

>- 对于远程服务：
>-  它只能隐式启动，类似隐式启动Activity，在清单文件中配置Service标签时，必须配置intent-filter子节点，并指定action子节点。启动时当然也要一一对应

例如：

		Intent intent = new Intent();
		intent.setAction("com.itheima.remote");
		stopService(intent);
or

		bindService(intent, conn, BIND_AUTO_CREATE);

### startService() 与 bindService()的混合调用  ###

>- 有时我们会有这种需求，服务必须要能独立运行与后台（不与activity共生死）， 且又需要利用bindService()来启动。
>- 这时 服务的启动必须要遵循一定的顺序， 不然会出问题！

->先开始、再绑定，先解绑、再停止


## 服务的生命周期相关方法 ##

>- 由于服务无前台界面，因此就不存在焦点及其相关方法。
>- 在第一次开启服务时会调用onCreate(） 和 onStartConmand(); 如果已经开启， 再次开启的话只会调用 onStartConmand()。

![](pictures/服务的生命周期.png)



## IntentService ##
>- 它是Service的一个子类。
> - IntentService使用队列的方式将请求的Intent加入队列，然后开启一个worker thread(线程)来处理队列中的Intent，
> - 对于异步的startService请求，IntentService会处理完成一个之后再处理第二个，每一个请求都会在一个单独的worker thread中处理，不会阻塞应用程序的主线程。
> 
> - 由于一般我们是不能再Service中进行一些耗CPU和耗时操作（下载什么的）， 这些耗时操作我们一般都会开启一个子线程来做。
> - 但是如果我们继承IntentService，由于这个类在处理事务时就是开启了线程来处理的，因此，我们可以将耗时操作直接放在服务里，就不用再手动开启子线程了。

	example：
	public class HelloIntentService extends IntentService {  
	  
	  /**  
	   * A constructor is required, and must call the super IntentService(String) 
	   * constructor with a name for the worker thread. 
	   */  
	  public HelloIntentService() {  
	      super("HelloIntentService");  
	  }  
	  
	  /** 
	   * The IntentService calls this method from the default worker thread with 
	   * the intent that started the service. When this method returns, IntentService 
	   * stops the service, as appropriate. 
	   */  
	  @Override  
	  protected void onHandleIntent(Intent intent) {  
	      // Normally we would do some work here, like download a file.  
	      // For our sample, we just sleep for 5 seconds.  
	      long endTime = System.currentTimeMillis() + 5*1000;  
	      while (System.currentTimeMillis() < endTime) {  
	          synchronized (this) {  
	              try {  
	                  wait(endTime - System.currentTimeMillis());  
	              } catch (Exception e) {  
	              }  
	          }  
	      }  
	  }  
	}


## 访问服务中的方法 ##
> - Service对象是由系统创建的（我们是弄不出来它的）。因此我们在activity中是不可能直接访问到- - Service的方法的。这可如何是好呢？
> - 为了访问到Service对象的方法，我们需要一个中间人对象-IBinder
> - 这个对象在服务的onBind()方法调用时会被返回。而在bindService()方法调用时，我们可以通过其ServiceConnection参数的onServiceConnected()方法拿到那个被返回的IBinder对象。
> - 原理就是->利用这个中间人IBinder来访问服务中的方法。

具体做法：


	public interface PublicBusiness {
	
		void QianXian();
	}

	public class LeaderService extends Service {
	
		@Override
		public IBinder onBind(Intent intent) {
			// 返回一个Binder对象，这个对象就是中间人对象
			return new ZhouMi();
		}
	
		class ZhouMi extends Binder implements PublicBusiness{
			public void QianXian(){
				banZheng();
			}
			
			public  void daMaJiang(){
				System.out.println("陪李处打麻将");
			}
		}
		
		public void banZheng(){
			System.out.println("李处帮你来办证");
		}
	}
	
	public class MainActivity extends Activity {
	
	    private Intent intent;
		private MyServiceConn conn;
		PublicBusiness pb;
		
		@Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	        intent = new Intent(this, LeaderService.class);
	        conn = new MyServiceConn();
	        //绑定领导服务
	        bindService(intent, conn, BIND_AUTO_CREATE);
	    }
	    
	    public void click(View v){
	    	//调用服务的办证方法
	    	pb.QianXian();
	    }
	
	    class MyServiceConn implements ServiceConnection{
	
	    	//连接服务成功，此方法调用
			@Override
			public void onServiceConnected(ComponentName name, IBinder service) {
				// TODO Auto-generated method stub
				pb = (PublicBusiness) service;
			}
	
			@Override
			public void onServiceDisconnected(ComponentName name) {
				// TODO Auto-generated method stub
				
			}
	    	
	    } 
	}


## 访问远程服务中的方法 ##
> - 上面的方式只能访问本地服务中的方法。这种方式就不能用来访问远程服务中的方法。
> - 这是因为，我们在上面  	pb = (PublicBusiness) service;  进行了强转，  而在不同的进程中你可能这样直接强转成功吗？？？？？
> 
> - 解决办法-AIDL（Android interface definition language）它用于进程间通信

使用步骤：

- 把远程服务的方法抽取成一个单独的接口java文件（这里叫他PublicBusiness）
- 把接口java文件的后缀名改成aidl
- 在自动生成的PublicBusiness.java文件中，有一个静态抽象类Stub，它已经继承了binder类，实现了PublicBusiness接口，这个类就是新的中间人
- 把这个生成的aidl文件复制到要进行访问这个服务的进程代码去，并且在复制时应注意aidl文件所在的包名必须跟提供服务项目中aidl所在的包名一致

关键：

	在访问远程服务项目中，强转中间人对象时，直接使用Stub.asInterface()进行强转：
		public void onServiceConnected(ComponentName name, IBinder service) {
			// TODO Auto-generated method stub
			PublicBusiness pb = Stub.asInterface(service);

		}


