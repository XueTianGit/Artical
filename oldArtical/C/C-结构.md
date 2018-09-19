title: C-结构
date: 2016/3/14 11:11:31      
categories: C
---

# C-结构 #

## 空结构体占用多大的内存？ ##
>使用sizeof测试时， 不同的编译器定义的值不同，
	但是现代续断编译器把它定义为只占一个字节，这样就避免了出现相同的地址。

## 由结构体产生柔性数组 ##
柔性数组：即数组的大小待定
原理：C于语言中结构体的最后一个元素可以是未知大小的数组。
范例：

	typedef struct _soft_array{
		int len;
		int array[];
	} SoftArray;
	
	int main()
	{
		SoftArray* sa = malloc(sizeof(SoftArray) + sizeof(int) * 10);
		sa->array[9] = 9;
		
		printf("%d", sa->array[9]);
		return 0;
	}

**NT: 在普通情况下， 你是不可能直接在C语言中这样写的。  int array[];**

## union的细节 ##

> - union只分配最大域的空间，所有域共享这个空间。
> - union的使用受系统大小端的影响， 看下面例子,你能确定C的值是几吗？

		union Demo{
			int i; 
			char c;
		};
		
		int main()
		{
			union Demo demo;
			demo.i = 1;
			
			printf("%d", demo.c);
		}

何为大小端？
> - 指的是字长大于8bit的处理器，在处理一个字的时候，将其拆分成多个字节的表示方法。
> - 对于大端处理器，高位在低地址，低位在高地址。如0x12345678，在内存中这样表示：
> - 地址 00 01 02 03
> - 数据 12 34 56 78
> - 一般，MIPS/PPC都是大端处理器。
> 
> - 而对于小端处理器，低位在低地址，高位在高地址。0x12345678这样排列：
> - 地址 00 01 02 03
> - 数据 78 56 34 12
> - x86属于小端。ARM默认是小端模式
		
## enumc常量 ##
- 1）enum定义的是真正的常量
- 2）enum可以使常量与有具体意义的名称联系在一起
- 3）enum默认常量在前一个值得基础上加1
- 4）enum类型只能取定义时的离散值
- 5）C允许对枚举变量使用运算符++（C++是不允许的）

	enum Month{
		Jay,Feb,Wes,Thur
	};

	enum Month month = Jay;
	month++;
	int a = Feb;
	printf("%d, %d", a, month);
	

enum与#define的区别：

- 1）#define宏常量只是进行简单的替换，枚举常量是真正意义上的常量。（你想在C中创造常量， 应该只有枚举可以）
- 2）#define宏常量无法被调试，但是枚举常量可以
- 3）#define宏常量无类型信息，枚举常量是一种带有特定类型信息的常量。

## typedef ##
- 1)typedef用于给一个已经存在的数据类型起一个别名
- 2）typedef并没有定义新的类型
- 3）typedef重定义的类型不能进行和unsigned 和signed扩展

typedef与#define的区别，看下面的这个例子：

	typedef char* PCHAR1;
	#define PCHAR2 char*

	PCHAR1 p1, p2;  //p1, p2 都是指针 - > char *p1, *p2; 
	PCHAR2 p3, p4;  //p3是指针，而p4不是-> char *p3, p4 






	

