title: OC-引用计数
date: 2016/2/29 15:45:57   
categories: Objective-C
---

# OC-引用计数 #

> 引用计数与内存管理息息相关

## 基础 ##
>- 原理非常简单
>    - 每个对象内部都维护着自己的引用计数
>    - alloc方法，将使对象的引用计数为1
>    - retain方法可以使对象的引用计数加1
>    - release方法可以使对象的引用计数减1
>    - 当对象的引用计数为0时，对象将被销毁，并返回给堆
>    - release方法不可以发送给错误的对象



- 所有权
>- OC中引用计数通常是依据所有权来讨论的。
>    - alloc出一个对象，你拥有这个对象
>    - 别人创建的对象，通过发retain消息，你可以获得这个对象的所有权
>    - 所有权并不是独立的，一个对象可有拥有多个对象的所有权，一个对象可有被多个对象持有
>    - 你如果拥有对象的所有权，不要忘记通过release消息来交出所有权
>    - 一个对象如果被别人“拥有”，那么它就会保持活动
>    - 如果没有被别人拥有，那么就会取消分配
>    - 通过调用"alloc, new, copy, mutableCopy"开头的方法来创建对象，你就拥有返回的对象

- dealloc
>- 当对象引用计数归0时，将会取消分配，这是我们应在dealloc做一些工作，以保证对象的引用计数正常，防止内存泄露
>- dealloc中[super dealloc]的调用非常重要
>- 最终会调用到NSObject对象的dealloc方法， 这个方法开启了一条事件链，它最终会调用系统库函数free()，将对象的内存返回给堆

	//一个RockStar对象的dealloc函数， 这个对象持有许多其他对象，比如guitar
	-(void) dealloc
	{
		[guitar release];
		//Release ant other objects held in instance variables
		
		[super dealloc];  //非常重要
		
	}

- retainCount
>- NSObject实现了retainCount方法，它用于返回一个对象的保留计数的当前值



## 自动释放 ##
>- 便利构造函数或者类似方法会面临一个问题
>    - 它会创建一个对象，但它无法跟踪那个对象，并在适合的时候发送release消息，来抵消对象的分配
>- 例如下面的类方法（便利构造函数） ： 

	+(Guitar*) guitar
	{
		Guitar* newGuitar = [[Guitar alloc] init];
		return newGuitar;
	}
>- 在这里 guitar 不是以"alloc, new, copy或mutableCopy"开头，因此调用者并不拥有它，无权发送release消息
>- 这个问题，OC利用自动释放机制（延迟释放）来解决

	+(Guitar*) guitar
	{
		Guitar* newGuitar = [[Guitar alloc] init];
		[newGuitar autorelease];
		return newGuitar;
	}


- 自动释放池
>- 可以使用@autorelease编译器指令以及一对大括号来创建自动释放池
>- 大括号定义类自动释放池的作用范围
>- 给对象发送一条autorelease消息，将把对象放在自动释放池中
>- 在程序突出自动释放池的作用域之后，通过autorelease消息放在池中的每个对象都会接受到一条release消息
>- autorelease消息返回接收者，即可以这样写

	SomeClass* autoreleaseObj = [[[SomeClass alloc] init] autorelease];
>- 注意点
>    - 可以给一个对象发送多条autorelease消息，这样这个对象在自动释放时，会接收到多条release消息
>    - 当程序退出自动释放作用域时，并不意味着池中的对象都会被取消分配， 只是池中的对象都会接受一条release消息而已

- 管理自动释放池
>- 运行库会维护一个自动释放池的栈，创建一个池将把它压到栈上，退出池的作用域则把它弹出栈，
>- 接收到autorelease消息的对象将被放在位于栈顶的池中。
>- 当栈上没有自动释放池时，对一个对象调用auorelease将会导致错误
> -对于GUI程序，AppKit或UIKit将会为你创建自动释放池
> -对于非GUI程序，你必须创建至少一个自动释放池

- 自动释放与IOS
>- IOS应用程序应避免为大对象使用自动释放
>- 这是因为，位于自动释放池的对象将计入程序的内存占用量，进而可能导致内存不足
>- 即，大对象能不使用便利构造函数就不使用，而时应该alloc出来，并在用完后释放

## NSZombie与保留循环 ##
- NSZombie
>- 即僵尸对象，使用它可以来跟踪对象的过度释放（调试）
>- 当启用NSZombie时，将不会把取消分配的对象返还给堆，将会把这些对象转变成NSZombie对象
>- NSZombie对象将记录发送给他们的任何消息
>- 当在Xcode中启用NSZombie特性后
>    - Xcode会显示出给取消分配的对象发送消息的任何位置
>    - Xcode还会显示最有可能成为问题源的代码

- 保留循环
>- 定义： 当两个或多个对象，互相持有对方的所有权时，将导致都不能被释放，最后导致内存泄露
>- 解决办法
>    - 修改代码，使得复制保留循环的对象中至少有一个对象不会保留另一个对象
>- OC中没有用于检测保留循环的自动方式， 骚年！ 阻止保留循环靠的是你自己的编码小心！！！！

