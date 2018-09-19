title: C-#与##
date: 2016/3/14 11:05:52   
categories: C
---


# C-#与## #

- Point1 (#)
	- #符号用作一个预处理运算符，可以把宏参数转换为字符串，
	>我们可能有下面这个需求
		
		#define Squre1(x) printf("The square of " #x" is %d \n", ((x)*(x)));
		#define Squre2(x) printf("The square of x is %d \n", ((x)*(x)));
	
		int main()
		{
			
			int y = 5;
		
			Squre1(y);  //打印结果是： The square of y is 25;	
			Squre2(y);	 // The square of x is 25;
			
						//即， 我们把宏参数，嵌入到了字符串中， 该过程称为参数字符窜化 
			return 0;
		}


- Point2 (##)
	- ##符号用于在编译器连接两个符号
	>例如：
		
		#define NAME(n)  name##n
		-> NAME(1)   //name1;    


	- 上面这个例子可能没什么意思，来看一下下面这个例子对##的妙用
	>你能理解吗？

		//利用##定义结构体类型
		#define STRUCT(type) typedef struct _tag_##type type;\
		struct _tag_##types
		
		
		STRUCT(Student){
			char* name;
			int id;
		};
		
		/*
		typedef struct _tag_Student Student; 
		struct _tag_Student{
			char* name;
			int id;	
		};
		
		*/
		
		
		int main()
		{
			Student s1;
			s1.name="小明";
			
			printf("%s", s1.name);
			return 0;
		}