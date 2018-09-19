title: OC-总览一
date: 2016/2/29 15:45:34  
categories: Objective-C
---


# OC-总览一 #

## 引言 ##
>- OC和C++一样，几乎完全继承自C语言
>- OC非常类似C++
>- OC基础的学习我是看的《Objective-C_2.0_Mac和iOS开发实践指南》， 这本书非常好，感觉讲的有点深
>- 下面我要慢慢的来消化一下这本书，以巩固OC基础语法与其特性


## 简单看一下 ##

- OC中的文件
>- .c .cc, .cpp .h: 分别是C语言的源文件， C++的源文件， 和头文件
>- .m ： OC源文件
>- .mm : OC++的源文件
>- .pl : Perl源文件
>- .o : 已经编译过的文件


### OC中的类 ###

- OC中的类的定义方法非常清晰(分离接口和实现文件)
> - 在 .h文件中利用@interface描述类的组成结构（外部可见的）
> - 在 .m文件中利用@implementation实现类中定义的方法

	//.h文件，  描述Person
	@interface Person : NSObject    //Person类继承自NSOBject， NSObject可以认为是OC中的类的上帝
	{
		NSString* name;             //定义一个实例变量name， 类型为NSString指针
		int age;
	}
	//get 与 set方法，  必须是这种形式
	-(void) setName: (NSString*) n;     
	-(void) name;       //应记住没有get      
	-(void) showInfo;   
	
	@end        //应以end结尾


	//.m 文件   实现文件中，一般都是对interface中描述的方法进行实现
	@implementation Person     
	
	-(void) setName: (NSString*) n
	{
		name = n;
	}
	
	-(NSString*) name
	{
		return name;
	}
	
	-(void) showInfo
	{
		NSSLog(@"My name is %@ and the age is %i", name, age);  //NSSLog 类似printf  . %@代表打印字符串， %i代表打印整型
	}
	@end	


- OC中的方法
>- 通过上面的例子， 可以简单的看出了OC中类的定义
>- 可以看出OC中的方法的书写格式是比较特别的， 下面来看一下OC中的方法是怎么来写的
例如：	
	-(void) setName: (NSString*) name;

> 一个一个的来看：

1. - 代表方法的类型， 是属于对象方法， 还是类方法(+)
2. (void) 返回值类型
3. setName 方法名
4. : 方法接收的参数
5. (NSString*) 参数类型
6. name 参数名（形参）

>- 在OC中可以对父类的方法进行重写
>- 在OC中不可以对方法进行重载！
>- OC中 self 类似与 this

## OC中的抽象类 ##
>- 在OC中没有用于抽象类的显示语法。
>- 如果具有一个类，他实际上就是一个抽象类，就应该在文档中指出这一点
>- 在OC中，无法阻止有人实际的实例化你认为是抽象的类


- 




- 


	

