title: IOS-KVC，KVO与MVC设计模式
date: 2016/3/9 8:47:45      
categories: IOS
---

# IOS-KVC，KVO与MVC设计模式 #

## IOS中的MVC开发 ##

### KVC（键值编码） ###

- 简单介绍
>- KVC是Key Value Coding的缩写，意思是键值编码。
>- 利用它我们可以通过类定义我们可以看到类的各种属性，那么使用属性的名称我们就能访问到类实例化后的对象的这个属性值。
>- 这个方法可以不通过getter／setter方法来访问对象的属性（即使是私有变量也可以）
>- 其实就是java中我们利用反射来动态读取类属性的值(BeanUtils还记得吗)，只不过在OC中帮我们完成了这个步骤

> - KVC的操作方法由NSKeyValueCoding.h头文件中提供
> - 这个头文件中定义了NSObject(NSKeyValueCoding)分类， 因此几乎所有对象都可以使用KVC
> - 常用方法如下
>     - 动态设置： setValue:属性值 forKey:属性名（用于简单路径）、setValue:属性值 forKeyPath:属性路径（用于复合路径，
>       例如Person有一个Account类型的属性，那么person.account就是一个复合属性）
>     - 动态读取： valueForKey:属性名 、valueForKeyPath:属性名（用于复合路径）
> - 这几个基本方法，是跟便捷的KVC方法的基础

	//举一个例子	
	//Account类， 实现为空
	@interface Account : NSObject	
	@property (nonatomic,assign) float balance;
	@end
	
	//Person类， showMessage的实现就是将 _age字段和name属性简单打印
	@class Account;
	
	@interface Person : NSObject{
	    @private
	    int _age;
	}
	@property (nonatomic,copy) NSString *name;
	@property (nonatomic,retain) Account *account;
	-(void)showMessage;
	@end
	
	//main方法
	int main(int argc, const char * argv[]) {
	    @autoreleasepool {
	        
	        Person *person1=[[Person alloc]init];
	        [person1 setValue:@"Kenshin" forKey:@"name"];
	        [person1 setValue:@28 forKey:@"age"];    //注意即使一个私有变量仍然可以访问，java中的暴力反射
	        
	        [person1 showMessage];	        //结果：name=Kenshin,age=28

	     
	        Account *account1=[[Account alloc]init];
	        person1.account=account1;//当然这一步骤也可以写成:[person1 setValue:account1 forKeyPath:@"account"];
									
	        [person1 setValue:@100000000.0 forKeyPath:@"account.balance"];  //访问复合属性
	        
	        NSLog(@"person1's balance is :%.2f",[[person1 valueForKeyPath:@"account.balance"] floatValue]);	       
		 //结果：person1's balance is :100000000.00
	    }
	    return 0;
	}

- KVC对属性的具体查找规则（假设现在要利用KVC对a进行读取）

>- 如果是动态设置属性，则优先考虑调用setA方法，如果没有该方法则优先考虑搜索成员变量_a,如果仍然不存在则搜索成员变量a，
>  则会调用这个类的setValue:forUndefinedKey：方法(注意搜索过程中不管这些方法、成员变量是私有的还是公共的都能正确设置)；
>- 如果是动态读取属性，则优先考虑调用a方法（属性a的getter方法），如果没有搜索到则会优先搜索成员变量_a，如果仍然不存在则搜索成员变量a
>  则会调用这个类的valueforUndefinedKey:方法(注意搜索过程中不管这些方法、成员变量是私有的还是公共的都能正确读取)；

### KVO(键值监听) ###

- 简单介绍
>- KVO其实是一种观察者模式，利用它可以很容易实现视图组件和数据模型的分离，
>  当数据模型的属性值改变之后作为监听器的视图组件就会被激发，激发时就会回调监听器自身。
>  在ObjC中要实现KVO则必须实现NSKeyValueObServing协议，不过幸运的是NSObject已经实现了该协议，因此几乎所有的ObjC对象都可以使用KVO。

>- 在ObjC中使用KVO操作常用的方法如下
>     - 注册指定Key路径的监听器： addObserver: forKeyPath: options:  context:
>     - 删除指定Key路径的监听器： removeObserver: forKeyPath、removeObserver: forKeyPath: context:
>     -  回调监听： observeValueForKeyPath: ofObject: change: context:  
>- KVO的使用步骤也比较简单：
>     - 通过addObserver: forKeyPath: options: context:为被监听对象（它通常是数据模型）注册监听器
>     - 重写监听器的observeValueForKeyPath: ofObject: change: context:方法

- 继续使用上面的原型为例
>-

	@implementation Person
	
	-(void)showMessage{
	    NSLog(@"name=%@,age=%d",_name,_age);
	}
	
	-(void)setAccount:(Account *)account{
	    _account=account;
	    //添加对Account的监听
	    [self.account addObserver:self forKeyPath:@"balance" options:NSKeyValueObservingOptionNew context:nil];
	}
	
	-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context{
	    if([keyPath isEqualToString:@"balance"]){//这里只处理balance属性
	        NSLog(@"keyPath=%@,object=%@,newValue=%.2f,context=%@",keyPath,object,[[change objectForKey:@"new"] floatValue],context);
	    }
	}
	
	-(void)dealloc{
	    [self.account removeObserver:self forKeyPath:@"balance"];//移除监听
	    //[super dealloc];//注意启用了ARC，此处不需要调用
	}
	@end
	
	//下面，在改变account属性时，就会调用observeValueForKeyPath: ofObject: change: context:方法

### MVC设计模式 ###

那么在IOS中如何进行MVC分层的呢？

#### .plist文件的读取 ####

>- .plist文件就是xml文件
>- xcode提供了对.plist文件很好的支持，我们可以很方便的对它进行操作
>- 我们可以把程序需要的简单数据存放在这个文件中
>- 根据MVC设计模式，数据是要被封装成模型的
>- 那么如何读取.plist文件内容（字典），将内容转成模型呢？

1. 定义模型
>-	

	//MJApp模型
	@interface MJApp : NSObject

	//名称
	@property (nonatomic, copy) NSString *name;

	//图标
	@property (nonatomic, copy) NSString *icon;
	
	/**
	 *  通过字典来初始化模型对象
	 *
	 *  @param dict 字典对象
	 *
	 *  @return 已经初始化完毕的模型对象
	 */
	- (instancetype)initWithDict:(NSDictionary *)dict;   //也提供一个成员方法
	
	+ (instancetype)appWithDict:(NSDictionary *)dict;  //提供一个类方法便于使用
	@end


	@implementation MJApp
	- (instancetype)initWithDict:(NSDictionary *)dict
	{
		//将字典中的数据，变成模型
	    if (self = [super init]) {
	        self.name = dict[@"name"];
	        self.icon = dict[@"icon"];
	    }
	    return self;
	}
	
	+ (instancetype)appWithDict:(NSDictionary *)dict
	{
	    return [[self alloc] initWithDict:dict];
	}
	@end


2. 定义模型对应的(视图)View，用来展示数据
>- 

	//  MJAppView.h
	@class MJApp;
	
	@interface MJAppView : UIView
	
	/**
	 *  模型数据
	 */
	@property (nonatomic, strong) MJApp *app;

	+ (instancetype)appView;	
	/**
	 *  通过模型数据来创建一个view
	 */
	+ (instancetype)appViewWithApp:(MJApp *)app;
	@end


	//MJAppView.m 文件
	@interface MJAppView() //通过扩展来隐藏内部控件
	@property (weak, nonatomic) IBOutlet UIImageView *iconView;
	@property (weak, nonatomic) IBOutlet UILabel *nameLabel;
	@end
	
	@implementation MJAppView

	+ (instancetype)appViewWithApp:(MJApp *)app
	{
	    NSBundle *bundle = [NSBundle mainBundle];
	    // 读取xib文件(会创建xib中的描述的所有对象,并且按顺序放到数组中返回)
	    NSArray *objs = [bundle loadNibNamed:@"MJAppView" owner:nil options:nil];
	    MJAppView *appView = [objs lastObject];
	    appView.app = app;
	    return appView;
	}
	
	+ (instancetype)appView  //创建没有数据的appView
	{
	    return [self appViewWithApp:nil];
	}
	
	- (void)setApp:(MJApp *)app
	{
	    _app = app;
	    
	    // 1.设置图标
	    self.iconView.image = [UIImage imageNamed:app.icon];
	    
	    // 2.设置名称
	    self.nameLabel.text = app.name;
	}
	
	@end

3. 控制器中传递数据给View进行展示
>-

	//  MJViewController.m
	
	@interface MJViewController ()
	/** 存放应用信息 */
	@property (nonatomic, strong) NSArray *apps;
	@end
	
	@implementation MJViewController
	
	//将.plist文件的内容转成MJApp模型， 并在恰当的时机设置给MJAppView
	- (NSArray *)apps   //利用getter方法，懒加载， 需要时再获取
	{
	    if (_apps == nil) {
	        // 初始化
	        
	        // 1.获得plist的全路径
	        NSString *path = [[NSBundle mainBundle] pathForResource:@"app.plist" ofType:nil];
	        
	        // 2.加载数组
	        NSArray *dictArray = [NSArray arrayWithContentsOfFile:path];
	        
	        // 3.将dictArray里面的所有字典转成模型对象,放到新的数组中
	        NSMutableArray *appArray = [NSMutableArray array];
	        for (NSDictionary *dict in dictArray) {
	            // 3.1.创建模型对象
	            MJApp *app = [MJApp appWithDict:dict];
	            
	            // 3.2.添加模型对象到数组中
	            [appArray addObject:app];
	        }
	        
	        // 4.赋值
	        _apps = appArray;
	    }
	    return _apps;
	}
	
	@end

