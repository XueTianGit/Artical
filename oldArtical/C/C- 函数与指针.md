title: C- 函数与指针
date: 2016/3/14 11:05:21  
categories: C
---


# C- 函数与指针#

- 基本点
	- 函数类型
		- C语言中函数有自己的特定类型
		- 函数的类型由返回值，参数类型和参数个数和参数顺序决定
		- int add(int i, char j) 的类型为  int(int, char)
	- typedef来定义函数类型
		- 语法： typedef type name(parameter list)
		- 例如 typedef int f(int, int)  ->  f为函数类型 int(int, int)
		-      typedef void p(int)     ->   p为函数类型 void(int)
	- 函数指针
		- 函数指针用于指向一个函数
		- 函数名是执行函数体的入口地址
		- 可以通过函数类型定义函数指针： FuncType* pointerFunc
		- 还可以间接定义函数指针：   type (*pointerFunc)(parameter, list)
	

> 通过下面程序可以看出：
> 
> - 函数的函数名代表一个地址
> - 在将函数名赋值给函数指针时  &FuncName 与 FuncName 没什么区别
> - 在利用函数指针调用函数时,  pf(); 与 (*pf)()  相同

	#include <stdio.h> 
	
	typedef int (Func)(int);
	
	int test(int i)
	{
		return i*i;
	}
	
	void f()
	{
		printf("Call f()...\n");
	}
	
	int main()
	{
		Func* pt = test; 
		void (*pf)() = &f;
		
		(*pf)();
		pf();
		
		printf("By Func's pointer call Function test() and result is %d\n", pt(2));
		
		return 0;
	}

- 回调函数
	- 回调函数时利用函数指针实现调用的一种机制
	- 为何要用回调函数（回调原理）
		- 调用者不知道具体时间发生的时候它调用什么样的函数
		- 被调函数也不知道会被谁调用，它只知道要完成一个任务
	- 可以看出，回调机制将调用者和被调函数分开， 进行了解耦


