title: Oracle-存储过程与存储函数
date: 2016/4/14 8:45:55  
categories: Database
---


# Oracle-存储过程与存储函数 #
存储在数据库中供所有用户程序调用的子程序叫存储过程、存储函数。

## 存储过程 ##
创建存储过程：

	create [or replace] procedure 过程名[(参数列表)]  
	as
	PLSQL程序体；【begin…end;/】

**注意的是：无declare**

有关于存储过程的参数问题：

	create procedure example(name in varchar2(10), result out varchar2(10)) 
	as
	...

	in代表输入参数， 可以省略不写。
	out代表输出参数， 存储过程想要定义返回值，out是必须写的。 

存储过程的调用语法：

	1）exec 存储过程
	2）在PLSQL程序块中调用
		范例：
		begin
		 raiseSalary(7369);
		end;
		/
	
	3）java程序调用
		