title: OC-控制结构
date: 2016/2/29 15:48:01        
categories: Objective-C
---

# OC-控制结构 #

>OC中的控制结构大部分继承自C,
>这里来看一下快速枚举和异常


- 快速枚举
>- 这个语法用于枚举集合
>- 它比基于NSEnumerator的传统循环运行的更快
>- 基本形式如下

	for(type loopVariable in expression)
	{
		//TODO
	}

>- expression必须求值为一个遵守NSFastEnumeration协议的对象
>- 功能类似与java的foreach循环
>- 需要注意的点
>    -   如果loopVariable是在for..in语句外面 声明的现有变量，它在for..in循环之后的值依赖于循环
>        是如何结束的，如果循环执行完毕，就会把loopVariable设置为nil，
>        但是，如果遇到break语句，loopVariable将保持在执行break语句时所具有的任何值
>    -   当然，在快速枚举的过程中是不允许改变集合的大小的。  

- 异常
>- 在OC中，利用编译器指令@try,@catch,@finally处理异常

	@try
	{
		int i = 1 / 0;
	}
	@catch(NSException* exc)
	{
		NSLog(@"Exception : %@ : %@", [exc name], [exc reason]);
	}
	@finally
	{
		NSLog(@"I don't want to say");
	}

>- 抛出自己的异常
>    - 可以使用@throw来抛出异常

	-(void) addOnion
	{
		if([pantry numOnions] <= 0)
		{
			NSException* exception = [NSException exceptionWithName:@"OutOfOnionException" reason:@"PantryEmpty" userInfo:nil];
			@throw exception;
		}
		else
		{
			@throw @"Again throwing a exception"
		}
	}
>- 用于捕获异常的@catch块必须具有与@throw语句使用的对象类型匹配的参数类型
>- 在Xcode中使用异常，必须开启异常选项

>- 应该使用异常吗
>    - 在OC中很少使用他们
>    - 更不推荐把异常用于流程控制工具
>    - 主要是因为，异常的发生会绕过我们期望执行的代码， 但是这些代码可能做很重要的工作呢！（比如清理内存）
	
	