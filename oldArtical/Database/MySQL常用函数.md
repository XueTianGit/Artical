title: MySQL数据库相关的函数
date: 2016/4/8 8:45:55  
categories: Database
---


#  MySQL数据库相关的函数 #

## 统计函数 ##
>count  
 
	count（m）  返回某一列，行的总数。
	example：
		select count(name) from student;
		select count(*) from student where math>80;
	NT: 对于count函数，他在统计是只统计有值的行

> sum 

	 用于求和
	example：
		select sum(math) from student;
		select sum(chinese)/count(*) from student;
		select sum(english), sum(math) from student;
	NT:sum 仅对数值起作用， 并且在进行多列求和时","是不能少的

>avg

	 用于求平均值
	example:
		select avg(math) from student

> max/min 

	 用于求最大值或最小值
	example：
		select max(chinese), min(chinese+math) from student;

## 时间日期相关的函数 ##
	DAY(date_expression)　 -- 函数返回date_expression中的日期值
	
	MONTH()　 --函数返回date_expression 中的月份值
	
	YEAR()　 --函数返回date_expression 中的年份值
	
	DATEADD(<datepart> ,<number> ,<date>) 
	　　--函数返回指定日期date 加上指定的额外日期间隔number 产生的新日期

> - ADDDATE(date,INTERVAL expr type) ADDDATE(expr,days) 
> - 当被第二个参数的INTERVAL格式激活后， ADDDATE()就是DATE_ADD()的同义词
> - mysql> SELECT DATE_ADD('1998-01-02', INTERVAL 31 DAY);           -> '1998-02-02'


> * DATEDIFF(expr,expr2) 
>   DATEDIFF() 返回起始时间 expr和结束时间expr2之间的天数。Expr和expr2 为日期或 date-and-time 表达式。计算中只用到这些值的日期部分。 
> 
> * now（）  返回当前时间
> * current_date(), current_time(), current_timestamp();


## 字符串相关的函数 ##
	ASCII()　　　　 --函数返回字符表达式最左端字符的ASCII 码值
	CHAR()　        --函数用于将ASCII 码转换为字符
	　　            --如果没有输入0 ~ 255 之间的ASCII 码值CHAR 函数会返回一个NULL 值
	LOWER()　       --函数把字符串全部转换为小写
	UPPER()　       --函数把字符串全部转换为大写
	STR()　         --函数把数值型数据转换为字符型数据
	LTRIM()　       --函数把字符串头部的空格去掉
	RTRIM()　       --函数把字符串尾部的空格去掉
	LEFT(),RIGHT(),SUBSTRING()　        --函数返回部分字符串
	CHARINDEX(),PATINDEX()            　--函数返回字符串中某个指定的子串出现的开始位置
	SOUNDEX()　                         --函数返回一个四位字符码C
	　　                                --SOUNDEX函数可用来查找声音相似的字符串但SOUNDEX函数对数字和汉字均只返回0 值
	DIFFERENCE()　                    　--函数返回由SOUNDEX 函数返回的两个字符表达式的值的差异
	　　--0 两个SOUNDEX 函数返回值的第一个字符不同
	　　--1 两个SOUNDEX 函数返回值的第一个字符相同
	　　--2 两个SOUNDEX 函数返回值的第一二个字符相同
	　　--3 两个SOUNDEX 函数返回值的第一二三个字符相同
	　　--4 两个SOUNDEX 函数返回值完全相同

## 数学相关的函数 ##