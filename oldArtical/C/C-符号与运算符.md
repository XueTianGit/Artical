title: C-符号与运算符
date: 2016/3/14 11:10:36       
categories: C
---

# C-符号与运算符#

## 注释 ##
> - 1）编译器会在编译的过程中删除注释， 但不是简单的删除而是以空格代替
> - 2）编译器会认为双引号括起来的内容都是字符串，双斜杠也不例外
> - 3）“/*..*/”注释不能被嵌套
> - 4）在编译器看来，注释和其他的程序元素都是平等的。


	y/*p；->
	这条语句会被当做注释，那么为什么呢？
	C在进行符号解析时，遵循贪心法则。即将/*作为一段注释的开始。

## 接续符和转移符 ##
> "\"作为接续符时，告诉编译器这样行尚未结束， 下一行继续写。
> 而编译器在解析接续符时会将反斜杠剔除， 跟在反斜杠后面的字符自动解引到上一行。

细节：

	1）在接续单词时，反斜杠后面不能有空格，不然，你就gbye了。
	2）接续符适合在定义宏代码块时使用
		#define SWAP(a, b)  \
		{				    \
			int temp = a;   \
			a = b;          \
			b = temp;       \
		}                   \
		int main()
		{
		
			int a = 3, b = 4l;
			SWAP(a, b)
			
			printf("%d, %d", a, b);
			return 0;
		}


"\"作为转移符时，主要用于表示无回显字符（代表一个动作），也可以用于表示常规字符：

	 ->  \ddd(1-3位八进制数代表的字符)    \xhh（1-2位十六进制数代表的字符）
		int main()
		{
			char a = '\018';
			char b = '\x1a';
			
			printf("demo : %d, %d", a, b);
			printf("dec = %d; octal = %#o; hex = %#x \n",  8, 8, 8);
		}

	`	结果：demo : 56, 26dec = 8; octal = 010; hex = 0x8

## 单引号与双引号 ##
->
>
	char* p1 = 1;
	char* p2 = '1';
	char* p3 = "1"; 

你能说出这些指针都指向哪里吗？

->
'a'表示字符常量，在内存中占一个字节，'a' + 1 表示'a'的ASCII码加1， 即'b'.
"a"表示字符串常量，在内存中占两个字节， "a" + 1表示的是指针运算， 结果为指向"a"结束符'\0'的指针。


->将字符串赋值给一个字符发生什么事?

	char c = "";   // ""代表内存的一个地址， 即把一个地址常量赋值给一个char类型的变量， 即会发生截断， 即将这个地址值得第八位赋值给c。









