title: OC-ARC
date: 2016/2/29 15:49:26            
categories: Objective-C
---

# OC-ARC #

## 基础 ##
>OC的内存管理确实使人非常的头疼，（我们不得不分散注意力在程序细节上！！！）
>并且，一旦出现内存管理问题，呵呵，往往就是程序挂掉。
>幸运的是，这个麻烦Apple帮我们解决了一大半 -> ARC

>- ARC代表自动引用计数，它可以自动为你插入 retain, release, autorelease消息
>    - ARC为OC对象管理内存，它不会管理Core Foundation对象或者利用malloc分配的原始字节
>    - ARC的工作是作为编译过程的一部分执行的， 
>    - ARC可以与利用手动引用计数编译的文件和库相互操作
>    - ARC不会查找和校正保留循环

- ARC的工作方式
>- ARC工作时，应用一组“本地规则”分析你的代码，并依据规则来插入retain和release消息
>- ARC在内存管理完之后，将会优化它已经管理的内存
>- ARC把保存OC对象的所有变量都视作强引用。
>- 在把一个对象赋予强引用变量时，ARC释放变量的当前内容，并保留新值
>- 当指向对象的最后一个强引用消失时，对象保留计数归0，取消分配，归还内存
>- ARC极其保守，它将比熟练的程序员插入更多的retain和release，然后依靠执行一遍高级优化来删除任何冗余信息

## 使用ARC的规则 ##

>为了可靠地做它工作，ARC会强加几个新规则

- 1.你不能自己调用内存管理方法（铜锣湾只能有一个浩南！）
>- 即在利用ARC编译的文件中，你不能发送任何retain，release，autorelease或retainCount消息
>- 这是因为，你如果发送这些消息，就代表你要与ARC争夺内存管理的控制权

- 2.ARC和dealloc
>- 在手动引用计数下，dealloc是你交出你的对象中保存的任何其他对象所有权的最后机会
>- 在ARC下将不再需要dealloc方法，ARC将负责释放保存在实例变量中的任何是对象
>- 但是，在ARC下，dealloc还是有点作用的
>    - 你自己用malloc分配的内存，你可能要在dealloc中释放
>    - 对于Core Foundation独享的引用或者关闭网络连接仍需要你自己来做
>- 在ARC下，你不再需要调用[super dealloc]，编译器将为你处理它，自己调用[super dealloc]将会导致编译错误

- 3.方法命名的约定
>- 回忆手动计数中的约定：
>    - 其名称以"alloc, new, copy, mutableCopy"开头的方法将返回具有 "+1的保留计数"的对象
>    - 你拥有这些方法返回的对象
>    - 如果对象是具有其他任何类型的名称的方法返回的，则不拥有他们
>- 在ARC中，这些约定变成了一个官方规则，即
>    - 名称以"alloc, new, copy, mutableCopy"开头的方法必须把所有权传递给调用代码，其他名称的方法不会传递所有权
>- 对于当整合ARC文件与非ARC文件时，ARC可以知道返回对象的状态的唯一方式是查看方法名
>- 如果你没有按这些规则，那么整合时可能就会有问题

- 4.ARC需要看到方法声明 
>- ARC需要看到它在消息表达式中遇到的每个方法的方法声明
>    - 即方法是在与消息表达式相同的文件中或者是在最终会导入那个文件的某个文件中声明的
>- 在手动引用计数下，使用其声明在消息表达式所在位置不可见的方法，如果方法真的存在的话，代码还是可以执行的
>- 在ARC下，看不到就是错误，你编译不过去的
>- 这是由于ARC不能通过方法名确定方法将返回一个对象，还是一种基本类型。

- 5.OC指针和C结构
>- 不能把指向OC对象的指针放在C结构或联合中。
>    - 这是因为ARC无法可靠的遵循C结构的生存期，因此它无法知道自己何时可以安全地释放赋予结构成员的任何对象
>    - 不过可以通过把保存对象的结构成员声明为_unsafe_unretained来解决这个问题
>        - _unsafe_unretained是在ARC中引入的新的变量修饰符，它告诉ARC不要对这个变量进行内存管理
>    - 其实更好的解决方案是把这个结构转变为类

	//error
	struct _sentence
	{
	//correct
	struct _sentence
	{
		__unsafe_unretained NSString* noun;
		__unsafe_unretained NSString* verb;
	} sentence;

## ARC中新的变量修饰符 ##

- __strong
>- 对象存储在一个__strong变量中，就创建了一个指向该对象的强引用
>- 当把一个对象赋予__stong变量时
>    - 首先将给该对象发送一条retain消息
>    - 如果该变量已经保存一个对象，ARC将给当前对象发送一条release或者autorelease消息
>    - 当指向对象的引用的__strong变量越出作用域时，该对象也将会收到一条release或者autorelease消息
>- 当最后一个指向对象的强引用消失时，对象的保留计数将归零，对象将被取消分配
>- 没有修饰符的变量默认都是__strong类型，因此我们很少会看到实际的编码__strong

	NSString* __strong employeeName;  //最好这样写
	//下面两种写法也是可以接受的
	NSString __strong* employeeName;  
	__strong NSString* employeeName;

- __weak
>- __weak变量实现了一个归零的弱引用，即
>    - 把对象存储在一个__weak变量中，ARC将不会保留它
>    - ARC也不会给变量中存储的以前内容发送一条release消息
>- “归零”的意义是
>    - 当保存在一个__weak变量中的对象由于指向他的最后一个强引用消失而取消分配时，将把__weak变量
>      以及指向对象的其他任何__weak引用都设置为nil
>    - 这是一个非常优秀的安全特性，它可以阻止你意外的给取消分配的对象发送消息并导致程序分配

- __autoreleasing
>- 当把一个对象赋予__autoreleasing变量时，ARC首先将保留该对象，然后自动释放它
>- 几乎很少使用它，它主要用于通过方法实参中的引用返回对象的变量
>    - Cocoa约定：通过引用返回对象不会把该对象的所有权转移给调用代码
>      通过引用返回的对象被期望将会自动释放

- __unsafe_unretained
>- 它实际上就是“赋值”的另一个名称，
>- 它意味着，把一个对象赋予被声明为__unsafe_unretained的变量时，ARC将不会进行内存管理
>- 可以理解为_unsafe_unretained是弱引用的一种形式，但是它是不安全的！！！

## 属性 ##
>- 可以把新的修饰符用于OC的声明式属性上，意义是相同的，并且不用加双下划线了
>- 在利用ARC编译过的属性声明中，strong和retain是同义词，assign和unsafe_unretained也是同义词

	//效果相同
	@property (nonatomic, strong) NSArray* anArray;
	@property (nonatomic, retain) NSArray* anArray;

## 保留循环和Core Foundation ##
- ARC不会校正保留循环
- ARC不会管理Core Foundation对象的内存

- ARC下OC对象与CF对象的免费桥接（强制类型转换）
>- 共分为3种类型
>    - (__bridge)告诉ARC在强制转换中不会转移所有权
>    - (__bridge_transfer)强制转换将把所有权从CF对象转移给ARC
>    - (__brdge_retained)它把所有权从ARC转移给CF

	NSString* nsString = (__bridge NSString*)cfString;
	
	//下面两句效果相同
	NSString* nsString = (__bridge_transfer NSString*)cfString;
	NSString* nsString = CFBridgingRelease(cfString);
	
	
	CFStringRef cfString = (__bridge_retained CFString*)nsString;
	CFStringRef cfString = CFBridgingRetain(nsString);

## 与void*之间来回进行强制转换 ##

>- ARC禁止在对象指针与类型化为void*的变量之间直接进行强制转换
>- 我们也不应该这么做！！

	void* cPointer = (void*)nsArray; //错误
	void* cPointer = (_bridge void*)nsArray;  //OK