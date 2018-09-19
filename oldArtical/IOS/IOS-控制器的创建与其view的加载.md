title: IOS-控制器的创建与其view的加载
date: 2016/3/9 8:50:08             
categories: IOS
---

# IOS-控制器的创建与其view的加载 #

## 控制器的创建 ##
>- 控制器的创建方式共有3种。
>- 应该明白的是控制器是要放到UIWindow上去的，因此应在应用程序代理的方法中创建，即
>    - -(BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions

1. 通过storyboard创建
>- 这时就是根据storyboard文件的描述来进行创建

	- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
	{
		//TODO1：创建window

		//先加载storyboard文件（Test是storyboard的文件名）
		UIStoryboard *storyboard = [UIStoryboard storyboardWithName:@"Test" bundle:nil];
		
		//接着初始化storyboard中的控制器
		//初始化“初始控制器”（箭头所指的控制器）
		MJViewController *mj = [storyboard instantiateInitialViewController];
		
		//通过一个标识初始化对应的控制器， 这个标志可以在storyboard文件中设置
		//MJViewController *mj = [storyboard instantiateViewControllerWithIdentifier:@”mj"];
	    
	    //TODO2：将控制器放到window上去
	    return YES;
	}

2.直接创建
>- 

	- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
	{
		//TODO1：创建window

	    MJOneViewController *one = [[MJOneViewController alloc] init];
	    one.view.backgroundColor = [UIColor blueColor];
		    
	    //TODO2：将控制器放到window上去
	    return YES;
	}

3.指定xib文件创建
>- 应明白的是
>    - xib文件中，File's owner属性对应的类应为要创建的控制器对应的类 (Custom Class)
>    - 控制器加载时要确定其view 
>        - 即应把对应的view的线给连上

	- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
	{
		//TODO1：创建window

	    MJThreeViewController *three = [[MJThreeViewController alloc] initWithNibName:@"MJThree5345" bundle:nil];
	    self.window.rootViewController = three;
			    
	    //TODO2：将控制器放到window上去
	    return YES;
	}

## 控制器view的创建 ##
> - 上面讨论的控制器的创建方式，那么控制的View是如何创建的呢？
> - 还是在 -(BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions方法中

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E6%8E%A7%E5%88%B6%E5%99%A8View%E7%9A%84%E5%88%9B%E5%BB%BA.png)
	
苹果官方控制器view的创建的创建方式

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E6%8E%A7%E5%88%B6%E5%99%A8view%E5%8A%A0%E8%BD%BD.png) 

>- 即
>    - 如果实现了loadView这个方法，则依据这个方法进行加载
>        - 因此，这个方法我们可以用来自定义控制器的View
>    - 如果是使用storyboard文件创建的控制器，则根据storyboard文件中的描述创建View
>    - 如果是使用xib文件创建的控制器，则根据xib文件中描述的view进行创建
>        - 如果 [[MJThreeViewController alloc] initWithNibName:nil bundle:nil];
>        - 即initWithNibName:nil 参数为nil，则会依次去寻找 MJView.xib， 和 MJViewController.xib文件
>        - 根据这两个文件进行view的创建
>        - 开发中，如果我们设置这个参数为nil， 则一般名字都命名为MJViewController.xib这种形式
>        - [[MJThreeViewController alloc] init];  也会去寻找这两个文件， 去进行view的创建

>- loadView的延迟加载
>    - 上面去加载view的步骤，只有在使用到控制器的view的时候才会去做
>    - 并且做完之后，会立即去调用 -(void)viewDidLoad 方法