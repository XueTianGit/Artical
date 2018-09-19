title: OC-类对象
date: 2016/2/29 15:47:02      
categories: Objective-C
---

# OC-类对象 #
>- 在OC中，类本身就是对象， 他们是类名为Class的特殊类的实例
>- 即你不必做任何事情来实例化类对象，比一期将通过类定义中的信息为你创建他们
>- OC中的类对象不具有实例变量，即无类变量
>- 但是，在OC中可以使普通的C语言的外部变量来模拟类变量
>- 我们可以这样使用类对象

	[SomeClass alloc];

- Class类型
>- 类型化为Class的变量用于指向类对象的指针， 可以这样获得这个指针

	Class aClass = [NSString class]; //Class天生就是一个指针类型
	Class aClass = [NSString superclass];

>- Classl类型变量可以代替类名作为类方法的接收者（那为什么，还要使用他呢？）

	//比如，你要创建一个对象，但是新对象的类直到运行时才能知道。
	NSString* className = ...;//需要读取配置文件
	Class classToInstantiate = NSClassFormString(className);
	id newObject = [[classToInstantiate alloc] init];

- 类方法
>- 类方法是在类的类对象上定义的方法，即类对象是类方法对应的消息接收者
>- 声明方式类似实例方法，例如： +(id) alloc;
>- 类方法中当然不能直接访问实例变量
>- 类方法可以被子类继承，子类可以重写类方法
>- 应注意： 类方法中的self指的是类对象，而不是类的实例

    - 便利构造函数、
	    - 即在单个方法中结合了分配和初始化，做用于类对象
	    	例如： NSString* emptyString = [NSString string];
		- 应注意的是： 你不拥有通过便利构造函数返回的对象

		- 便利构造函数创建的一般规范：
			+(id) dog
			{	
				//self, 保证子类也可以很好的使用这个方法
				return [[[self alloc] init] autorelease];  //即你你不拥有通过便利构造函数返回的对象
			}
		- 如果计划，获取便利构造函数返回的对象，你就要得到他的所有权
			Dog* dog = [Dog dog];
			[dog retain];
		
- 类的初始化
>- 第一次使用类对象时(向类对象发送alloc消息时)，会引起initialize消息的调用
>- 我们可以重写这个方法，一般，我们是不需要去重写的
>- initialize方法是有问题的
>    - 若父类重写的initialize方法，而子类没用重写
>    - 则父类的initialize方法可能会被两次调用，分别为：第一次使用父类， 第一次使用子类
>    - 我们应通过判断， 将初始化仅限于父类

	+(void) initialize
	{
		if( self == [Dog class] )
		{
			//TODO
		}

	}

- 模拟类变量
>- OC中并不存在类变量， 但是向类变量这个玩意，一般我们是有需求的。
>- 由于OC继承自C， 因此我们可以使用C的方法来模拟类变量
>    - 通过在类的实现文件中使用普通的文件作用域变量来模拟类变量。
>    - 并定义类方法，来获取与设置类变量
>- 但是，这种做法是有缺点的
>   - 首先，这个变量不会被继承（子类，要想有一个与父类一样的静态变量，要自己定义，并且名字不能相同）
>   - 当然，也不能像java中那样： 类名.变量 似的调用