title: IOS-多线程(一)
date: 2016/3/9 8:50:08             
categories: IOS
---

# IOS-多线程(一) #

>讲到多线程就离不开队列、线程池、锁等话题，下面来看一下IOS中的有关多线程的API

## 简述 ##

在IOS中共有3中多线程技术：

>- NSThread 
>     - 使用NSThread对象建立一个线程非常方便
>     - 但是！要使用NSThread管理多个线程非常困难，不推荐使用
>     - 技巧！使用[NSThread currentThread]跟踪任务所在线程，适用于这三种技术
>- NSOperation/NSOperationQueue
>     - 是使用GCD实现的一套Objective-C的API
>     - 是面向对象的线程技术
>     - 提供了一些在GCD中不容易实现的特性，如：限制最大并发数量、操作之间的依赖关系
>     - 它是Apple现在非常推荐使用的多线程技术的API
>- GCD —— Grand Central Dispatch (大中央调度)
>     - 是基于C语言的底层API（一听就很快）
>     - 用Block定义任务，使用起来非常灵活便捷
>     - 提供了更多的控制能力以及操作队列中所不能使用的底层函数

## GCD ##

大中央调度(想到了中央空调.....)

>- GCD的基本思想是就将操作(block)放在队列(queue)中去执行
>- 操作使用Blocks定义
>- 队列负责调度任务执行所在的线程以及具体的执行时间
>- 队列的特点是先进先出(FIFO)的，新添加至对列的操作都会排在队尾
>- GCD的函数都是以dispatch(分派、调度)开头的


- 队列
>- 在gcd中队列是dispatch_queue_t类型的
>- 队列中存放着一个一个的操作(block)
>- 队列分为串行队列和并行队列
>    - 串行队列：队列中的任务只会顺序执行
>    - 并行队列：队列中的任务通常会并发执行

- 队列中操作的派发
>- 派发方式分为两种
>    - dispatch_async(异步操作)：操作会并发执行，无法确定任务的执行顺序
>    - dispatch_sync (同步操作): 会依次顺序执行，能够决定任务的执行顺序

### 串行队列 ###
	
	//创建一个串行队列, “susion”对队列起一个标识作用
	dispatch_queue_t q = dispatch_queue_create("susionSerial", DISPATCH_QUEUE_SERIAL);


	//将操作添加到串行队列中并使串行队列同步方式操作进行派发
	//在派发的过程中会保证： 必须前一个操作执行完毕，才会去派发下一个操作(即是有顺序的！！)
	//不会去开辟一条新的线程执行队列中的任务，会直接在当前线程执行任务(这里是主线程)
	dispatch_sync(q, ^{
	    NSLog(@"串行同步任务一 %@", [NSThread currentThread]);
	});  


	//将操作添加到串行队列中并使串行队列异步方式操作进行派发
	//串行队列会以异步的方式对操作进行派发，但是由于要确保顺序: 会开辟出一条新的线程来执行队列中的操作
	//这里是直接在主线程中开辟一条子线程执行操作(但还是有顺序的!!)

    for (int i = 0; i < 10; ++i) {
	    dispatch_async(q, ^{
	        NSLog(@"%@ %d", [NSThread currentThread], i);
	    });
    }

	//非ARC开发时，千万别忘记release
	//dispatch_release(q);


### 并行队列 ###

	  dispatch_queue_t q = dispatch_queue_create("susionConcurrent", DISPATCH_QUEUE_CONCURRENT);

		//并行队列异步派发任务
		for (int i = 0; i < 10; ++i) {
	        // 异步任务
	       dispatch_async(q, ^{
	           NSLog(@"%@ %d", [NSThread currentThread], i);
	       });
	    }
	    
		//并行队列同步派发任务
	    for (int i = 0; i < 10; ++i) {
	        // 同步任务顺序执行
	        dispatch_sync(q, ^{
	            NSLog(@"%@ %d", [NSThread currentThread], i);
	        });


### 串行队列与并行队列的总结与对比 ###

- 串行队列
   - 串行队列，同步任务，不需要新建线程
   - 串行队列，异步任务，需要一个子线程。（“是最安全的一个选择”，应用场景一般是：既不影响主线程，又需要顺序执行的操作！）
- 并行队列	
   - 并行队列，同步任务，不需要新建线程，  
   - 并行队列，异步任务，有多少个任务，就开N个线程执行，
- 同步任务
   - 没有创建线程的欲望
   - 必须一个任务执行完才能执行另一个
- 异步任务
   - 有创建线程的欲望，但是创建多少条线程，取决与队列的类型

>- 即队列的类型决定了操作是否顺序执行，同步与异步决定了是否开启线程来执行队列中的操作
>- 对于并行队列同步任务，最主要的应用就是：阻塞并行队列任务的执行，只有当前的同步任务执行完毕后，后续的任务才能够执行
>- 对于一个队列中的操作，你可以以多种不同的形式派发！！！比如上面这句话的场景
		

### 主队列 ###
>- 和Android相同,Apple要求所有对UI的更新操作都应该在主线程中进行
>    - 这主要是因为UIKit框架中的对象并不是线程安全的
>- 我们一般更新UI的操作放到主队列中执行
>- 主队列中的操作都应该在主线程上顺序执行的，即不存在异步的概念，但是你应该异步添加！！
>- 由于主线程是死循环，即果把主线程中的操作看成一个大的Block，那么除非主线程被用户杀掉，否则永远不会结束
>- 所有主队列中添加的同步操作永远不会被执行，会死锁

	//这个队列中的任务会在主线程中执行，一般使用他来更新UI
	dispatch_queue_t q = dispatch_get_main_queue();
	
	//应该这样向主队列中派发任务
	dispatch_async(q, ^{
   		NSLog(@"主队列异步 %@", [NSThread currentThread]);
	});

	//这样添加的任务是不会被执行的，因为它在等待主线程的操作执行完，但是主线程是执行不完的
	dispatch_sync(q, ^{
    	NSLog(@"主队列同步 %@", [NSThread currentThread]);
	}); 

### 全局队列 ###

>- 苹果为了方便多线程的设计，提供一个全局队列，供所有的APP共同使用， 感觉就不好
>- 全局队列与并行队列的区别
>  - 不需要创建，直接GET就能用
>  - 两个队列的执行效果相同
>  - 全局队列没有名称，调试时，无法确认准确队列
    
    // 记住：在开发中永远用DISPATCH_QUEUE_PRIORITY_DEFAULT
    // 多线程的优先级反转！低优先级的线程阻塞了高优先级的线程！
    dispatch_queue_t q =dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

### 不同队列中嵌套dispatch_sync的结果 ###

	// 全局队列，都在主线程上执行，不会死锁
	dispatch_queue_t q = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
	// 并行队列，都在主线程上执行，不会死锁
	dispatch_queue_t q = dispatch_queue_create("susion", DISPATCH_QUEUE_CONCURRENT);
	// 串行队列，会死锁，但是会执行嵌套同步操作之前的代码， 
	//理解：（你正在执行，但是你又同步派发一个任务，你派发的这个同步任务会等你执行完(又不开新的线程)，而你又在等待你派发的这个任务执行完，这就蛋疼了）
	dispatch_queue_t q = dispatch_queue_create("susion", DISPATCH_QUEUE_SERIAL);
	// 主队列，直接死锁
	dispatch_queue_t q = dispatch_get_main_queue();
	
	dispatch_sync(q, ^{
	    NSLog(@"同步任务 %@", [NSThread currentThread]);
	    dispatch_sync(q, ^{
	        NSLog(@"同步任务 %@", [NSThread currentThread]);
	    });
	});

### dispatch_sync的应用场景 ###
>阻塞并行队列的执行，要求某一操作执行后再进行后续操作，如用户登录

	dispatch_queue_t q = dispatch_queue_create("cn.itcast.gcddemo", DISPATCH_QUEUE_CONCURRENT);
	__block BOOL logon = NO;
	dispatch_sync(q, ^{   //先在同步派发一个任务，把这个任务执行完再说！！！！
	    NSLog(@"模拟耗时操作 %@", [NSThread currentThread]);
	      [NSThread sleepForTimeInterval:2.0f];
	    NSLog(@"模拟耗时完成 %@", [NSThread currentThread]); 
	    logon = YES;
	});
	
	dispatch_async(q, ^{  //OK， 你可以并行了
	       NSLog(@"登录完成的处理 %@", [NSThread currentThread]);
	});


### GCD小结 ###
GCD

- 通过GCD，开发者不用再直接跟线程打交道，只需要向队列中添加代码块即可
- GCD在后端管理着一个线程池，GCD不仅决定着代码块将在哪个线程被执行，它还根据可用的系统资源对这些线程进行管理。
  从而让开发者从线程管理的工作中解放出来，通过集中的管理线程，缓解大量线程被创建的问题
- 使用GCD，开发者可以将工作考虑为一个队列，而不是一堆线程，这种并行的抽象模型更容易掌握和使用

GCD的队列

- GCD公开有5个不同的队列：运行在主线程中的主队列，3 个不同优先级的后台队列，以及一个优先级更低的后台队列（用于 I/O）
- 自定义队列：串行和并行队列。自定义队列非常强大，建议在开发中使用。在自定义队列中被调度的所有Block最终都将被放入到系统的全局队列中和线程池中
- 提示：不建议使用不同优先级的队列，因为如果设计不当，可能会出现优先级反转，即低优先级的操作阻塞高优先级的操作

GCD队列示意图

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOSgcd%E9%98%9F%E5%88%97%E7%A4%BA%E6%84%8F%E5%9B%BE.png)





  






