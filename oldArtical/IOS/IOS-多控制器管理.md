title: IOS-多控制器管理一
date: 2016/3/9 8:49:52             
categories: IOS
---

# IOS-多控制器管理一 #

## 控制器的生命周期方法 ##

	/**
	 *  view加载完毕
	 */
	- (void)viewDidLoad
	{
	    [super viewDidLoad];
	}
	
	/**
	 *  view即将显示到window上
	 */
	- (void)viewWillAppear:(BOOL)animated
	{
	    [super viewWillAppear:animated];
	}
	
	/**
	 *  view显示完毕(已经显示到窗口)
	 */
	- (void)viewDidAppear:(BOOL)animated
	{
	    [super viewDidAppear:animated];
	}
	
	/**
	 *  view即将从window上移除(即将看不见)
	 */
	- (void)viewWillDisappear:(BOOL)animated
	{
	    [super viewWillDisappear:animated];
	}
	
	/**
	 *  view从window上完全移除(完全看不见)
	 */
	- (void)viewDidDisappear:(BOOL)animated
	{
	    [super viewDidDisappear:animated];
	}
	
	/**
	 *  view即将销毁的时候调用
	 */
	- (void)viewWillUnload
	{
	    [super viewWillUnload];
	}
	
	/**
	 *  view销毁完毕的时候调用
	 */
	- (void)viewDidUnload
	{
	    [super viewDidUnload];
	    
	    // 由于控制器的view已经不在了,需要显示在view上面的一些数据也不需要
	    self.apps = nil;
	    self.persons = nil;
	}
	
	/**
	 *  当接收到内存警告的时候
	 */
	- (void)didReceiveMemoryWarning
	{
	    [super didReceiveMemoryWarning];
	
	}

>- 需注意的是： viewWillUnload与viewDidUnload方法，现在基本不用覆写
>    - 只有在非ARC的环境下我们才需要覆写这两个方法
>    - 一般在viewDidUnload方法中释放view相关的资源
>- 当应用接收到内存警告时，会引起这两个方法的调用

![](C:\Users\Administrator\Desktop\TheLastTaskOf\博客的html文件\IOS\图片\内存警告处理.png)

## 多控制器管理 ##

>- 控制器的管理类似与View管理View
>- 即给众多需要管理的控制器找一个爹，这个爹其实还是一个控制器
>- 在IOS7中提供了两个这样的角色： UINavigationController和UITabBarController

### UINavigationController ###

- 代码管理
>- UINavigationController中维护着一个控制器栈
>    - 显示在导航控制器上的永远是栈顶控制器
>    - @property(nonatomic,copy) NSArray *viewControllers;
> 	   @property(nonatomic,readonly) NSArray *childViewControllers;
>- 使用push方法能将某个控制器压入栈
>    - -(void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated;
>- 使用pop方法可以移除控制器
>    - 将栈顶的控制器移除
>	   - (UIViewController *)popViewControllerAnimated:(BOOL)animated;
>    - 回到指定的子控制器
>      - (NSArray *)popToViewController:(UIViewController *)viewController animated:(BOOL)animated;  
>    - 回到根控制器（栈底控制器）   
>      - (NSArray *)popToRootViewControllerAnimated:(BOOL)animated;

>- 修改导航栏的内容
>  - 应知道：一个导航控制器只有一个导航栏，导航栏的内容由栈顶控制器的navigationItem属性决定
>  - 导航栏的高度为44
>  - 坐标起点为(0, 20)  , 20为状态栏的高度
>  - 栈顶控制器的View为全屏显示

>- UINavigationItem有以下属性影响着导航栏的内容
>     - 左上角的返回按钮（其实影响的是下一个控制器左上角的显示）
>       @property(nonatomic,retain) UIBarButtonItem *backBarButtonItem;
>     - 中间的标题视图
>       @property(nonatomic,retain) UIView          *titleView;
>     - 中间的标题文字
>       @property(nonatomic,copy)   NSString        *title;
>     - 左上角的视图
>       @property(nonatomic,retain) UIBarButtonItem *leftBarButtonItem;
>     - UIBarButtonItem *rightBarButtonItem  右上角的视图
>       @property(nonatomic,retain) UIBarButtonItem *rightBarButtonItem;

- storyboard中利用UINavigationController进行管理
>- 在storyboard文件中进行管理是十分方便的
>    - 可以直接看出整个app有多少的个控制器和控制器之间的跳转关系
>    - 直接通过拖线操作，即可完成push
>    - 可以很方便的修改状态栏
>- 不要往回拖线！！！！

#### 导航栏 ####

- 有关导航栏的背景图片的尺寸
>- IOS6上
>    - 非retina:320*44; retina:640*88
>- IOS7上
>    - retina: 640*128  （128是因为把状态栏给同化了）

- 设置导航栏和BarButtonItem的主题
>- 
	// 1.设置导航栏主题
	UINavigationBar *navBar = [UINavigationBar appearance];
	// 设置背景图片
	NSString *bgName = @"NavBar";
	[navBar setBackgroundImage:[UIImage imageNamed:bgName] forBarMetrics:UIBarMetricsDefault];
	
	// 设置标题文字颜色
	NSMutableDictionary *attrs = [NSMutableDictionary dictionary];
	attrs[NSForegroundColorAttributeName] = [UIColor whiteColor];
	attrs[NSFontAttributeName] = [UIFont systemFontOfSize:16];
	[navBar setTitleTextAttributes:attrs];
	
	// 2.设置BarButtonItem的主题
	UIBarButtonItem *item = [UIBarButtonItem appearance];
	// 设置文字颜色
	NSMutableDictionary *itemAttrs = [NSMutableDictionary dictionary];
	itemAttrs[NSForegroundColorAttributeName] = [UIColor whiteColor];
	itemAttrs[NSFontAttributeName] = [UIFont systemFontOfSize:14];
	[item setTitleTextAttributes:itemAttrs forState:UIControlStateNormal];

## 控制器间的跳转 ##
- storyboard中的跳线
>- storyboard上每一根用来界面跳转的线，都是一个UIStoryboardSegue对象（简称Segue）
>    - 每一个Segue对象，都有3个属性
>        - 唯一标识
>         @property (nonatomic, readonly) NSString *identifier;
>       - 来源控制器
>         @property (nonatomic, readonly) id sourceViewController;
>       - 目标控制器
>         @property (nonatomic, readonly) id destinationViewController
>- Segue的类型
>    - 根据Segue的执行（跳转）时刻，Segue可以分为2大类型
>        - 自动型：点击某个控件后（比如按钮），自动执行Segue，自动完成界面跳转
>        - 手动型：需要通过写代码手动执行Segue，才能完成界面跳转


- 自动型Segue
>- 一般用于界面跳转间并没有什么逻辑判断时使用
>- 按住Control键，直接从控件拖线到目标控制器
>- 如果点击这个控件后，不需要做任何判断，一定要跳转到下一个界面

- 手动型Segue
>- 一般用于界面跳转间并没有什么逻辑业务要做
>- 按住Control键，从来源控制器拖线到目标控制器
>- 手动型的Segue可以在Interface Builder中设置一个标识， 用于我们在代码中寻找
>- 手动型Segue必须由源控制器执行
>    - [self performSegueWithIdentifier:@"login2contacts" sender:nil]; （self为源控制器）

- performSegueWithIdentifier:方法进行界面跳转的过程
>1. 根据identifier去storyboard中找到对应的线，新建UIStoryboardSegue对象
>    1. 设置Segue对象的sourceViewController（来源控制器）
>    2. 新建并且设置Segue对象的destinationViewController（目标控制器）
>2. 调用sourceViewController的下面方法，做一些跳转前的准备工作并且传入创建好的Segue对象
>    - -(void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender; 
>3. 调用Segue对象的- (void)perform;方法开始执行界面跳转操作
>    - 如果segue的style是push
>      取得sourceViewController所在的UINavigationController
>      调用UINavigationController的push方法将destinationViewController压入栈中，完成跳转
>    - 如果segue的style是modal
>      调用sourceViewController的presentViewController方法将destinationViewController展示出来

- Modal
>- 上面UINavigationController中控制器的切换方式是push
>- Modal是另一种控制器的切换方式
>- Modal的默认效果：新控制器从屏幕的最底部往上钻，直到盖住之前的控制器为止
>- 跳转方法
>    - 以Modal的形式展示控制器
>      -(void)presentViewController:(UIViewController *)viewControllerToPresent animated: (BOOL)flag completion:(void (^)(void))completion
>    - 关闭当初Modal出来的控制器
>      -(void)dismissViewControllerAnimated: (BOOL)flag completion: (void (^)(void))completion;
>- 需要知道的是
>    - 在使用Modal进行控制器跳转时，窗口的根控制器是不会变化的，只是暂时的移到一边
>    - 如果使用A控制器Modal出B控制器，则A控制器的presentViewController属性的值为B














