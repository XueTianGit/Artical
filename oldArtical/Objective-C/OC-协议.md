title: OC-协议
date: 2016/2/29 15:46:15    
categories: Objective-C
---

# OC-协议 #

## 基础 ##
>- 协议是一个类可以选择实现的一组定义的方法(类似java中的接口)
>- OC中协议分为：正式协议和非正式协议
>- 正式协议： 协议中的方法都要实现
>- 非正式协议： 可以选择实现

- 声明协议
>- 协议的声明位于头文件中
>- 协议没有对应的实现文件
>- 在OC2.0之后允许把协议方法标记为可选或必须的
>    - 采用协议的类必须实现协议的所有必须方法
>    - @required是默认的

	@protocol DrawableItem
	
	@required
	-(void) drawItem;
	-(NSRect) boundingBox;
	-(void) setColor:(NSColor*)color;

	@optional
	-(NSString)labelText;
	@end


- 采用协议
>- 一个类可以通过在@interface行中添加用尖括号括住的协议名称，来告诉编译器它采用哪个协议
>- 如果一个类采用协议或者他继承自摸个采用协议的类，那么这个类就必须遵守协议
>- 一个类可以采用多个协议：在单独一组尖括号中列出这些协议，并用逗号分隔他们
>- 协议的顺序是不重要的

 	@interface AcmeUniversalBolt: AcmeComponent<DrawableItem, NSCopying>

- 协议作为类型
>- 声明一个变量时，可以通过向类型声明中添加协议名称，把变量声明为一种可以保存遵守协议的对象的类型

	id<DrawableItem> currentGraphic; //currentGraphic所指向的对象必须遵守DrawableItem协议

- 属性和协议
>- 如果某些协议方法是访问器方法，就可以使用属性语句在协议头文件中声明他们
>- 采用这种协议的任何类都必须合成与属性对应的访问器，或者提供他们的实现， 就向在类的@interface部分声明属性一样

	@protocol DrawableItem
	-(void) drawItem;
	@property (nonatomic, retain) NSString* name;	
	@end

- NSObject类和NSObject协议
>- 既有NSObject类，也有NSObject协议
>- NSObject是几乎所有的OC对象的根类
>- NSObject协议是一个正式协议，其中列出了任何类为成为良好的OC“公民”而必须实现的方法。
>- NSObject类采用NSObject协议，然后大多数类都将直接或者间接的从NSObject继承这些方法
>- Foundation的另一个根类是NSproxy(用于构建分布式系统)， 也采用NSObject协议

- 协议对象
>- 协议对象是代表协议的对象，他们是Protocol类的成员
>- 利用@protocol()指令从协议的名称中获得协议对象。
>- 与类对象不同，协议对象没有方法
>- 他们只能被用于为NSObject类的conformsToProtocol:方法的参数
>- 这个方法可以用于判断接收者是否实现了协议中所有必须的方法
>- 即，可以使用这个方法来判断一个对象(或者类)是否遵守协议、

	Protocol myDataSourceProtocol = @protocol(TablePrinterDataSource);
	
	//判断对象或者类是否遵守协议
	if( [newDataSource conformsToProtocol: @protocol(TablePrinterDataSource)] )
	{
		//TODO
	}
	
	or
	
	if([Person conformsToProtocol: @protocol(TablePrinterDataSource)])
	{
		//TODO
	}


- 非正式协议
>- 说白了就是没有实现方法的类别
>- 如果一个类别只有头文件却没有实现，编译器是不会报错的
>- 但是应注意的是：
>    - 在编码采用非正式协议的类时，协议的方法实现必须位于类的@implementation部分，而不是类别的@implementation部分
>    - 如果把这些方法放在单独的类别@implmentation部分，则是将把所有方法实现添加到所有类中！！！！