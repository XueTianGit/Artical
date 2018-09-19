title: IOS-多线程(二)
date: 2016/3/9 8:50:08             
categories: IOS
---

# IOS-多线程(二) #

>继续看

## NSOperation & NSOperationQueue ##
简介

- NSOperationQueue(操作队列)是由GCD提供的队列模型的Cocoa抽象，是一套Objective-C的API
- GCD提供了更加底层的控制，而操作队列则在GCD之上实现了一些方便的功能，这些功能对于开发者而言通常是最好最安全的选择
- 从本质上来看，操作队列的性能会比GCD略低，不过，大多数情况下这点负面影响可以忽略不计，操作队列是并发编程的首选工具

队列及操作

- NSOperationQueue有两种不同类型的队列：主队列和自定义队列
	- 主队列运行在主线程上
	- 自定义队列在后台执行
- 队列处理的任务是NSOperation的子类
	- NSInvocationOperation
	- NSBlockOperation

基本使用步骤

- 定义操作队列
- 定义操作
- 将操作添加到队列
- 注意：一旦将操作添加到队列，操作就会立即被调度执行


>- 把操作直接添加到自定义队列中，默认会开启多条线程来执行Operation
>- 可以对队列设置最大线程数(但是不能保证)
>    - [queue setMaxConcurrentOperationCount: 3]

### NSBlockOperation(块操作) ###
>NSBlockOperation十分灵活

	//定义操作队列
	myQueue = [[NSOperationQueue alloc] init];

	定义操作并添加到队列
	NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
	    //TODO
	}];
	将操作添加到队列
	[myQueue addOperation:op];

### NSInvocationOperation(调度操作) ###
>看到Invocation就应该知道，你要准备一个被调用的函数

	操作调用的方法
	- (void)operationAction:(id)obj  (接收的参数就是下面的object参数)
	{
	    NSLog(@"%@ - obj : %@", [NSThread currentThread], obj);
	}
	....

	定义操作并添加到队列	
	NSInvocationOperation *op = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(operationAction:) object:@(i)];
	[myQueue addOperation:op];  //使用上面的那个队列


### 为操作增加顺序性 ###
>可以通过添加操作依赖来使操作之间产生顺序关系

	NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"%@ - 下载图片", [NSThread currentThread]);
	}];
	NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
	    NSLog(@"%@ - 添加图片滤镜", [NSThread currentThread]);
	}];
	NSBlockOperation *op3 = [NSBlockOperation blockOperationWithBlock:^{
	    NSLog(@"%@ - 更新UI", [NSThread currentThread]);
	}];
	[op2 addDependency:op1];
	[op3 addDependency:op2];
	[myQueue addOperation:op1];
	[myQueue addOperation:op2];
	[[NSOperationQueue mainQueue] addOperation:op3];
	
>- 可以看出
>    - 利用addDependency可以指定操作之间的彼此依赖关系(执行先后顺序)
>    - 操作是可以跨队列的！！！(很牛)

## NSObject的多线程方法 ##
>- NSObject的多线程方法使用的是NSThread的多线程技术
>- NSThread的多线程技术不会自动使用@autoreleasepool
>    - 即它在开启线程，执行任务时并不会自动释放任务
>    - 因此，如果在后台执行很多任务时，他会开启很多线程
>    - 线程是很耗电和耗内存的，因此在这一点上就不好
>-  主线程中是有自动释放池的，使用GCD和NSOperation也会自动添加自动释放池

	//开启后台执行任务的方法
	//我们可以在后台执行的方法中更新UI，但是不应该这么做！！！
	-(void)performSelectorInBackground:(SEL)aSelector withObject:(id)arg

	//在后台线程中通知主线程执行任务的方法
	//(BOOL)wait这个参数： YES会阻塞主线程，直到调用方法完成， NO不会阻塞主线程，主线程会继续执行
	-(void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait

	//获取线程信息
	[NSThread currentThread]
	//线程休眠
	[NSThread sleepForTimeInterval:2.0f];

## RunLoop ##

- Run Loop提供了一种异步执行代码的机制，不能并行执行任务
- 在主队列中，Main Run Loop直接配合任务的执行，负责处理UI事件、计时器，以及其它内核相关事件
- Run Loop的主要目的是保证程序执行的线程不会被系统终止


- Run Loop的工作特点
	- 当有事件发生时，Run Loop会根据具体的事件类型通知应用程序做出响应
	- 当没有事件发生时，Run Loop会进入休眠状态，从而达到省电的目的
	- 当事件再次发生时，Run Loop会被重新唤醒，处理事件

- 主线程和其他线程中的Run Loop
	- iOS程序的主线程默认已经配置好了Run Loop
	- 其他线程默认情况下没有设置Run Loop
	- 一般在开发中很少会主动创建RunLoop，而通常会把事件添加到RunLoop中


![](C:\Users\Administrator\Desktop\TheLastTaskOf\博客的html文件\IOS\图片\UIApplication中的RunLoop.jpg)


## 多线程与循环引用 ##
>- 一般来说，如果我们在一个block中使用了self类似的变量，self又持有这个block，那么就会造成循环引用
>- 但是在多线程中，单纯在操作对象中使用self不会造成循环引用
>- 这是因为，当线程销毁时，不管操作对象(block)中有什么,都是直接销毁掉

## 解决共享资源访问问题的锁 ##
>- 在OC中共有两种锁：互斥锁和原子锁

- 互斥锁
>类似与java中的synchonized语句块
	@synchronized(self)
	{
	}

- 原子锁
> 对属性的访问进行加锁，是细粒度的所，这个锁允许同时读，但不允许同时写
	@property (atomic, strong)NSString* name;

- 小结
>- 既然使用了锁，那么程序性能肯定是要下降的(代价昂贵)
>- 为了保证性能，atomic仅针对属性的setter方法做了保护
>- 而争抢共享资源时，如果涉及到属性的getter方法，可以使用互斥锁@synchronized可以保证属性在多个线程之间的读写都是安全的
>- 解决共享资源并发访问的根源肯定是：减少共享资源的个数！


## OC中的单例 ##
>- 由于在OC中是不允许对属性或者结构成员直接进行初始化的，所以OC中的单例不存在饿汉式

- 懒汉式单例的实现
>- 重写 allocWithZone：方法， 保证分配唯一的对象
>- 提供sharedXXX或者mainXXX获得单例对象
>- 使用dispatch_once来解决懒汉式的并发分配问题
>    - dispatch_once( dispatch_once_t *predicate, dispatch_block_t block);
>      其中第一个参数predicate，该参数是检查后面第二个参数所代表的代码块是否被调用的谓词，
>      第二个参数则是在整个应用程序中只会被调用一次的代码块。dispach_once函数中的代码块只会被执行一次，而且还是线程安全的。
	
	@implementation DemoObj
	
    // static DemoObj *instance;  //可以理解为类变量
	// 在iOS中，所有对象的内存空间的分配，最终都会调用allocWithZone方法
	+ (id)allocWithZone:(struct _NSZone *)zone
	{
	    static DemoObj *instance; 
	    static dispatch_once_t onceToken;   	// GCD提供了一个方法，专门用来创建单例的
	    dispatch_once(&onceToken, ^{           //onceToken默认为0
	        instance = [super allocWithZone:zone]; 	        // 在多线程环境下，永远只会被执行一次，instance只会被实例化一次
	    });
	    
	    return instance;
	}
	
	+ (instancetype)sharedDemoObj
	{
		return instance = [[self alloc] init];
	}


	
