title: OC-对象
date: 2016/2/29 15:48:44          
categories: Objective-C
---

# OC-对象#

## OC中创建对象 ##
>- 创建OC对象需要两个步骤： 分配和初始化
>- 最后会返回指向完成对象的指针
OC中创建对象有两种方法：
	1. [[Person alloc] init]  //在创建对象时，这两部一般合成在一块， 以防止alloc分配不成功，而初识化类野指针
	2. [Person new]    //就是合并了上面的两步， 当时在初始化时并不能传递参数


>- allloc 与 allocWithZone
>- alloc默认是在堆上分配内存，而allocWithZone 则可以从指定区域分配内存
>- alloc被实现为 allocWithZone:nill
>- 限制Apple不推荐使用allocWithZone


- 构造方法 init
>- 一个类一般都有多个构造方法，都是以init开头
>- 典型写法
	-(id) init   //id 可以被理解为时 (void*)类型
	{
		if(self = [super init])   //保证父类初识化
		{
			/*对本类做初始化工作*/
		}
		
		return self;
	}

>- 带有多个参数的初始化器    

	-(id) initWithShirtSize:(NSUInteger) inShirtSize color:(NSColor*) inShirtColor
	{
		if( self = [super init] )
		{
			shirtSize = inShirtSize;
			shirtColor = [inShirtColor retain];  //必须要保留这个对象的
		}
		
		return self;
	}

- 指定的初始化器（就是为了保证类的完全初始化，的口头约定）
>- 首先，在OC语法中并没有指定初始化器的正式概念。
>- 指定初始化器是将完全初始化类的实例的初始化器（不调用它就初识化不成功对象）
>    - 所有其他的本类初始化器最终必须调用本类的指定初始化器
>    - 本类的指定初识化器必须调用其超类的指定初识化器

- 失败的初识化
>- 如果在初始化对象的过程中失败了， 我们应该这么做
>    - 执行没有被dealloc处理的任何清理工作
>    - 释放self，（这是由于self是由alloc利用保留计数1创建的， 必须利用release抵消alloc）
>    - 返回nill
>- 下面来看一下，拥有两个数组实例变量的类是如何初识化的

	-(id) init
	{
		if( self = [super init] )
		{
			anArray1 = [[NSMutableArray] init];
			
			if( anArray1 == nill )
			{
				[self release];
				return nil;
			}
			
			anArray2 = [[NSArray alloc] initWithContentOfFile: @"/Users/rclair/stuff"];
			
			if( anArray2 == nil )
			{
				[self release];
				return nil;
			}
		}
		
		return self;
	}

>在执行[self release]之后， self中保存的对象将不再有效， 因此在对self调用release与返回nil之间，不应该执行
>任何引用self的代码。

## 对象的销毁 ##
>- 在OC中， 不会显示销毁对象
>- 当对象使用完毕后， 可以给他发送一条release消息， 如果对象的保留计数变为0， 那么对象的dealloc方法会被自动调用
>- dealloc类似C++中的析构函数
>- 在dealloc中我们应
>    - 给正在消失的对象创建或保留的任何对象发送一条release消息
>    - 调用超类的dealloc方法  （很重要， 并且应该出现在最后一行）


## 复制对象   
>- NSObject提供类一个copy方法， 但是这个copy方法并没做任何实现
>- 允许复制的类，采用NSCopying协议
>- 这意味着， 想要自己的类可以被复制，应实现 copyWithZone: 方法。

- 浅复制和深复制
>- 浅复制： 把原始的指针对象简单的提供给新的对象
>- 深复制： 依次复制任何子对象，并把子对象的副本也提供给新对象
>    - 可以看出， 浅复制将副本和原始对象纠缠在一起
>    - 深复制将会产生两个完全独立的对象

- 可变复制与不可变复制
>- NSArray, NSDictory, NSset这些对象都是不可变的实例。
>- 不过，可以通过这些实例，来复制出可变实例

	NSMutabkeArray* aMutableCopyOfAnArray = [anArray mutableCopy];

>- 对于不可变的类，复制很简单
>- 由于独享是不能被修改的， 因此原始对象与副本之间永远不会有任何区别， 因此copyWithZone可以这样实现

	-(id) copyWithZone:(NSZone*)zone
	{
		return [self retain];
	}