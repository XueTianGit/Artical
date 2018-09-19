title: OC-属性
date: 2016/2/29 15:46:38     
categories: Objective-C
---
# OC-属性 #

## 基础 ##
>- 在OC中获取器与设置器的命名规范是： instanceVariableName和 setInstanceVariableName
>- 手写访问器当然是非常麻烦的，毕竟也没什么技术含量
>- OC2.0引入了一个称为"声明的属性(属性)"的新特性，它添加了两条新的语句，减少了我们的编码工作
>    - @property：提供了一种声明访问器方法的简写方式
>    - @synthesize：使编译器能够为你编写指定的访问器方法
>    - @dynamic： 可以用来告诉编译器，访问器方法我自己提供，不用你帮忙了

使用方法：

	//使用@property语句编码的Employee.h
	#import <Foundation/Foundation.h>
	
	@interface Employee : NSObject
		NSInteger employeeNumber;
		NSString* name;
		Employee* suoervisor;
		NSInteger salary;
	@end
	
	//使用@peoperty语句替换访问器方法的声明， 单独一行就可以同时声明获取器和设置器
	@property (nonatomic, readonly) NSInteger employeeNumber;
	@property (nonatomic, retain) NSString* name;
	@property (nonatomic, retain) Employee* suoervisor;
	@property (nonatomic, assign) NSInteger salary;
	
	
	//Employee.m
	#import "Employee.h"
	@implementation Employee
		//@synthesize 使得编译器为你创建访问器方法
		//也可以这样写：@synthesize employeeNumber,name,suoervisor,salary
		@synthesize employeeNumber;
		@synthesize name;
		@synthesize suoervisor;
		@synthesize salary;
		
	@end

>- 注意
>    - 如果你使用了@synthesize合成了访问器方法，但是又自己提供了访问器或者设置器。
>      这时编译器会以你的访问器为优先者，并且缺少哪一个访问器就替你补足哪一个访问器

## 其他须知 ##

- 实例变量和属性可以具有不同的名称
>- 为属性和他表示的实例变量使用不同的名称是可能的，但是你必须在@synthesize语句中通知编译器

	//这时， 合成的访问器依然以 propertyName 为准
	@synthsize propertyName = instanceVariableName

- 指定访问器的名称
>- 正常情况下，合成的名称肯定是遵守规范的
>- 但也有一些情况下，我们可能不想让合成的方法名称遵守规范
>- 比如对于BOOL属性，按常规方式合成可能比较别扭，我们可以这样做

	//获取器的名称将为：isCaffeinated
	@property (nonatomic, getter=isCaffeinated) BOOL caffeinated;

- 合成实例变量
>- 其实上面的代码也是有点麻烦是不是？
>- 现在，我们可以这样做：即创建了实例变量，也提供了他的访问器方法
>- 但是，这种方式只适合于现代的运行库
>- 并且：只能在类的主实现部分或者在与类的主实现部分相同的.m文件中编码的类别或者子类中直接访问合成的实例变量

	@interface Employee : NSObject
		@property (nonatomic, retain) NSString* name;  //直接合成了实例变量: name	  
	@end
	
	@implementation Employee
 		@synthesize name;
 	@end

- 默认使用@synthesize
>- 在Xcode4.4以前，用于属性合成的默认编译器指令是@dynamic
>    - 如果没有为属性显示编码@synthesize和@dynamic，编译器将假定使用@dynamic， 即期望你提供访问器方法
>- 在Xcode4.4之后
>    - 属性合成的默认编译器指令是@synthesize
>    - 即，他将为你自动合成访问器方法和实例变量
>    - 但是，合成的实例变量的名字为：下划线+属性名
>    - 即相当于：@synthesize myProperty = _myProperty


- 合成总结
>- @dynamic:一切我自己来
>- @synthesize:
>    - 编译器将合成你没有自己编码的任何访问器，如果你没有显示声明一个实例变量，那么将帮你合成（注意名字） 

- 类扩展中的声明属性
>- 这些属性我们不让别人看到，我们自己用！！
>- 即可以在实现文件中， 这样做：

	@interface MyClass()
		@property (nonatomic, retain) NSString* name;
		...
	@end

- dealloc
>- 在没有使用ARC的情况下，dealloc还是要你自己来完成的
>- 在dealloc中，你应该根据你的属性的合成的情况，释放对象的所有权，并调用父类的dealloc

## 细看@property ##
>- 属性声明的形式为：
>    - @property (attributes) type name;

- attributes
> - 它将影响如何构造合成的设置器
> - 分为三组（三种）
>     - assign,retain,copy 
>     - readwrite,readonly
>     - nonatomic

>- assign
>    - 设置器将简单的把新值赋予属性
>    - assign特性是非对象属性所允许的唯一选择
>    - 对于引用计数下的对象属性，assign将创建一个弱引用
>    - 不指定默认为assign
>- retain
>    - 将保留对象的所有权

	-(void) setSupervisor: (Employee*)newSupervisor
	{
		[newSupervisor retain];
		[suoervisor release];
		suoervisor = newSupervisor;
	}
	
>- copy
	- 复制对象，释放原对象
	
	-(void) setName: (NSString*)newName
	{
		NSString* oldName = name;
		name = [newName copyWithZone: nil];
		[oldName release];
	}

>- readwrite与readonly
>    - readwrite： 两个访问器都合成
>    - readonly： 只合成获取器
>- nonatomic
>    - 属性默认都是原子(atomic)的，即线程安全
>    - nonatomic即指明属性不考虑线程安全问题

## 点语法 ##
>- 在OC2.0中还引入了一种新语法，称为点语法， 它是用于调用访问器方法的简写语法
>- 例如，对于上面的Employee的访问器方法我们可以这样调用】
>- 但是，点语法并不依赖于属性
>    - 知道C的都知道，C的结构的方法和成员变量，就是通过点语法来访问的
>    - 点语法可能会和C结构的访问方式混淆

	NSInteger currentSalary = employee.salary;  //就是调用的获取器
	employee.salary = 1000000;  //就是调用的设置器