title: IOS-IOS应用的核心一
date: 2016/3/9 8:47:09    
categories: IOS
---

# IOS-IOS应用的核心一 #

对比于android开发，两者的相似程度还是非常高。

## IOS应用的核心类 ##

- UIApplication
>- 类似android中的Application对象，代表整个应用程序
>- 单例的
>- 主要用于从系统接收事件，并将事件分派的应用自定义代码中进行处理
>    - 其内部拥有一个 id<UIApplicationDelegate> _delegate 字段
>- 它是应用程序启动创建的第一个对象
>- 利用UIApplication对象，能进行一些应用级别的操作
>    - 设置应用程序图标右上角的红色提醒数字
>      > @property(nonatomic) NSInteger applicationIconBadgeNumber;
>    - 设置联网指示器的可见性（旋转菊花）
>      > @property(nonatomic,getter=isNetworkActivityIndicatorVisible) BOOL networkActivityIndicatorVisible;
>    - 管理状态栏
>      - 从iOS7开始，系统提供了2种管理状态栏的方式
>      - 通过UIViewController管理（每一个UIViewController都可以拥有自己不同的状态栏）
>      - 通过UIApplication管理（一个应用程序的状态栏都由它统一管理）
>      - 在iOS7中，默认情况下，状态栏都是由UIViewController管理的， 管理办法是通过方法的实现
>      - 不同点显而易见
>      - 因此， 如果想利用UIApplication来管理状态栏，首先得修改Info.plist的设置（View controller-based status bar appearence属性）
>- UIApplication有个功能十分强大的openURL
>    - -(BOOL)openURL:(NSURL*)url;
>    - 使用这个方法，我们可以完成许多事情

		openURL:方法的部分功能有
		打电话
		UIApplication *app = [UIApplication sharedApplication];
		[app openURL:[NSURL URLWithString:@"tel://10086"]];
		
		发短信
		[app openURL:[NSURL URLWithString:@"sms://10086"]];
		
		发邮件
		[app openURL:[NSURL URLWithString:@"mailto://12345@qq.com"]];
		
		打开一个网页资源
		[app openURL:[NSURL URLWithString:@"http://ios.itcast.cn"]];
		
		打开其他app程序

- 遵守UIApplicationDelegate协议的类
>- 从上面的UIApplication对象我们也可以知道
>    - 使用这个协议，我们可以接收UIApplication对象传递给我们的应用程序事件
>        - 应用程序的生命周期事件
>        - 状态栏改变相关事件
>        - 远程和本地通知相关事件 
>        - 一些系统事件等等
>    - 比如下面这些方法


	// app接收到内存警告时调用
	- (void)applicationDidReceiveMemoryWarning:(UIApplication *)application;
	// app进入后台时调用（比如按了home键）
	- (void)applicationDidEnterBackground:(UIApplication *)application;
	// app启动完毕时调用
	- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions;
>- 其中含有一个UIWindow对象的引用

- UIView
>- 类似Android中View的存在，代表界面

- UIWindow
> - UIWindow是一种特殊的UIView，通常在一个app中只会有一个UIWindow
> - iOS程序启动完毕后，创建的第一个视图控件就是UIWindow，接着创建控制器的view，最后将控制器的view添加到UIWindow上，
>   于是控制器的view就显示在屏幕上了
> - 一个iOS程序之所以能显示到屏幕上，完全是因为它有UIWindow
> - 也就说，没有UIWindow，就看不见任何UI界面
> - 该对象被UIApplicationDelegate对象引用
> - 即我们可以自己创建并设置给应用程序委托

	/**
	 *  程序启动完毕就会调用一次
	 */
	- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
		// 1.创建window
	    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
	
	    // 2.设置window的背景色
	    self.window.backgroundColor = [UIColor whiteColor];
	    
		//设置Window的rootViewController
	    MjOneViewController *one = [[MjOneViewController alloc] init];
		//   [self.window addSubview:one.view];  //这种设置是有问题的，因为window对象没有保留控制器对象，
		//控制器对象会很快销毁
	    //self.window.rootViewController = one;
	
	    // 
	    [self.window makeKeyAndVisible];  //是window成为主窗口并可见
	    return YES;
	}

>- 我们可以这样获取应用程序中的window对象

	[UIApplication sharedApplication].windows
	在本应用中打开的UIWindow列表，这样就可以接触应用中的任何一个UIView对象
	(平时输入文字弹出的键盘，就处在一个新的UIWindow中)
	
	[UIApplication sharedApplication].keyWindow
	用来接收键盘以及非触摸类的消息事件的UIWindow，而且程序中每个时刻只能有一个UIWindow是keyWindow。
	如果某个UIWindow内部的文本框不能输入文字，可能是因为这个UIWindow不是keyWindow
	
	view.window
	获得某个UIView所在的UIWindow
	
	需要知道的是，系统键盘在弹出时，也是存在一个window对象上的 (UITextEffectsWindow)

- UIViewController
>- 个人目前感觉，他就像android中的Activity，可以使用它来管理界面布局，协调和应用其他部分的关系
>- 该对象需要被添加到UIWindow上才能显示
>- 其下有一个View对象，这个对象就是其所管理的界面


- IOS应用程序的启动流程
>- 看程序启动，当然要把main函数拿过来看一下啦

	int main(int argc, char * argv[])
	{
	    @autoreleasepool {
	        return UIApplicationMain(argc, argv, nil, NSStringFromClass([MJAppDelegate class]));
	    }
	}
>- 可以看出关键就在UIApplicationMain函数的4个参数
>    - 前两个不用说了
>    - 第三个参数类型为NSString* principalClassName， 
>        - 它用于指定应用程序类名，该类必须是UIApplication的子类
>        - 如果为nil，则使用UIApplication类作为默认值
>    - 第四个参数类型为 NSString* delegate, 顾名思义是应用程序委托对象的类名
>        - 该类必须遵守UIApplicationDelegate协议
>- UIApplicationMain()函数是不会返回的，其内部有一个死循环，它时刻监听系统事件，并分发给应用程序代理
>- 通过以上可以看出
>    - 我们必须给UIApplication设置一个应用程序代理对象
>    - 应用程序代理对象必须拥有一个UIWindow对象
>    - UIwindow对象必须（也可以不必须）有一个ViewController对象
>    - ViewController那下面肯定有它管理的View视图界面
>    - 这样应用程序才能完整启动

- 上面这4大对象的关系如下图

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS4%E5%A4%A7%E6%A0%B8%E5%BF%83%E5%AF%B9%E8%B1%A1%E7%9A%84%E5%85%B3%E7%B3%BB.png)




 
