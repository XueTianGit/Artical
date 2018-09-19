title: OC-类别与扩展
date: 2016/2/29 15:47:22       
categories: Objective-C
---

# OC-类别与扩展 #

## 类别 ##
>- 类别可以让你在现有的类中添加额外的方法，而不必子类化它，也不必访问类的源代码。
>- 使用类别来扩展类比使用子类来扩展类轻松多了
>- 一个给定的类，可以具有多个类别，但类别不能重名
>- 类别中的方法和本类中的方法是平等的，即可以被继承，可以访问本类资源

这里以给NSString类扩展一个camelCase()方法为例：

		//NSString+CameCase.h
		#import <Foundation/Foundation.h>
		
		@interface NSString(CamelCase)
			-(NSString*) camelCaseString;
		@end
		
		//NSString+CameCase.m
		#import "NSString+CameCase.h"
		
		@implementation NSString(CamelCase)
		
			-(NSString*) camelCaseString
			{
				//TODO
			}
		@end

>- 我们这里定义了一个类别： NSString(CamelCase)
>- 定义一个类别，就像定义一个类一样，规范形式是分别建.h和.m文件（文件命名规范一般为： 类+类别.h）
>- 在@interface描述方法， 在@implementation进行实现
>- 需要注意
>    - 类别头文件必须导入用于所扩展的类的头文件
>    - 类别中不能向类中添加任何实例变量

- 利用类别重写方法
>- 使用类别可以重写类的方法，但应注意
>    - 一旦重写，新版本将替换旧版本
>    - 类别版本无法调用原始方法
>    - 不能使用一个类别的方法重写另一个类别的方法，这并不会导致编译错误，但是最终使用的版本依赖于加载顺序

## 关联引用 ##
>- 使用关联引用，可以使我们将一个对象与另一个对象关联起来，即相当于把一个对象贴到另一个对象身上
>- 一个源对象可以有多个关联引用，只要每个关联引用都具有它自己唯一的键即可
>- 关联引用是单向引用，源对象知道它具有关联的值，但是值对象并不知道他是关联的一部分

下面代码在UIImage对象和NSString对象之间创建了一个关联

	#import <objc/runtime.h>
	
	static char captionKey;
	
	UIImage* myImage = ....  //get a image
	NSString* caption = ...  //
	
	objc_setAssociatedObject{
		myImage,         //参数一： 源对象，即添加关联的对象
		&captionKey,     //用于关联的键(key)， 这个建被类型化为void*, 一般使用静态变量的地址
		caption,         //与源对象关联的值
		OBJC_ASSOCIATION_COPY_NONATOMIC   //关联策略
	};

>- 关联策略
>   - 关联策略确定源对象是保留关联对象，在创建关联是复制它，还是只保存关联对象，而不会保留或复制它
>     关联策略还确定创建关联是否是一个原子操作
>   - 常见的关联策略有
>   	- OBJC_ASSOCIATION_COPY_NONATOMIC
> 		- OBJC_ASSOCIATION_ASSIGN
> 		- OBJC_ASSOCIATION_RETAIN_NONATOMIC
> 		- OBJC_ASSOCIATION_RETAIN
>  		- OBJC_ASSOCIATION_COPY


- 检索关联
>- 可以通过使用键来获得与键对应的关联对象

	NNString* caption = objc_getAssociatedObject(image, &captionKey);

> 删除关联 	

	objc_setAssociatedObject(image, &captionKey, nil, OBJC_ASSOCIATION_ASSIGN);  //将键对应的对象设置为nil
	//or
	objc_removeAssocaitedObjects(image);  //直接删除了一个对象的全部关联

## 扩展 ##
>- 扩展可以使我们在类的实现文件中添加一个接口部分， 即在公共视图之外声明方法、属性和实例变量
>- 我们通过扩展定义的方法、属性和变量外部当然是不可以使用的
>- 可以把扩展看成一个匿名的类别，区别是
>    - 扩展方法的实现必须位于相同文件的@implementation部分
>    - 与类别的情况不同，编译器将为你执行检查；如果在扩展中声明类了一个方法但是忘记实现它，那么你将得到一个警告

例如：这里我们给Person类，扩展一个方法 isGay()，我们可以这样做:
	//Person.m文件
	#import "Person.h"
	
	@interface Person()
		-(BOOL) isGay;
	@end
	
	@implementation Person
		-(BOOL) isGay
		{
			NNSLog(@"YES?");
			return YES;
		}
	@end