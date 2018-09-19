title: Oracle-常用通用函数、条件判断函数和多行函数
date: 2016/4/18 8:45:55  
categories: Database
---


# Oracle-常用通用函数、条件判断函数和多行函数 #

## 通用函数  ##
> 通用函数就是可以作用于任何类型的函数（参数类型可以是number或varchar2或date类型）

	1）NVL(a,b)
	如果a为null值，则取b为返回结果，否则返回a。
	
	2）NVL2(a,b,c)
	如果a为null， 则结果为b， 否则结果为c。
	
	3）NULLIF(a,b)
	在类型一致的情况下，如果a与b相同，返回NULL，否则返回a。
	例如：比较10和10.0是否相同
	select NULLIF(10,'10') from dual;   ->错误， 类型不一致


## 条件判断函数 ##

1）case表达式
> case表达式是SQL99标准。一个CASE表达式的默认返回值类型是任何返回值的相容集合类型，但具体情况视其所在语境而定。
> 如果用在字符串语境中，则返回结果味字符串。如果用在数字语境中，则返回结果为十进制值、实值或整数值。  

	语法1：
	SELECT 
	case 字段 
	     when 条件1 then 表达式1
	     when 条件2 then 表达式2
	     else 表达式n
	end 
	
	语法2：
	SELECT 
	CASE
	WHEN 条件 THEN 表达式1 ELSE 表达式2 END;

	范例：
	SELECT CASE 1 WHEN 1 THEN 'one'
	WHEN 2 THEN 'two' ELSE 'more' END;   -> 结果为“one”
	
	SELECT CASE WHEN 1>0 THEN 'true' ELSE 'false' END;          -> 'true'
	
	具体范例： 查询emp表，将职位是分析员的，工资+1000；职位是经理的，工资+800；职位是其它的，工资+400
	select ename "姓名",job "职位",sal "涨前工资",
	       case job
		    when 'ANALYST' then sal+1000
		    when 'MANAGER' then sal+800
	            else sal+400
	       end "涨后工资"
	from emp; 


![](C:\Users\Administrator\Desktop\TheLastTaskOf\博客的html文件\Database\图片\条件判断函数1.png)

2）decode()函数
> decode()函数是专属oracle的语法

	语法：
	decode(字段,条件1,表达式1,条件2,表达式2,...表达式n)
	
	还是上面那个问题， 现在利用decode()函数来解决：
	select ename "姓名",job "职位",sal "涨前工资",
	       decode(job,'ANALYST',sal+1000,'MANAGER',sal+800,sal+400) "涨后工资"
	from emp; 
	结果相同。


## 多行函数 ##
> 多行函数：输入多个参数，或者是内部扫描多次，输出一个结果，例如：count(*)->（count在计算结果时，扫描了14次呢）
> 并且注意的是， 在oracle中，多行函数是不会统计null值的。

1）count

	范例1：
	统计emp表中员工总人数
	select count(*) from emp;
	***号适用于表字段较少的情况下，如果字段较多，扫描多间多，效率低，项目中提倡使用某一个非null唯一的字段，通常是主键 **
	
	范例2：
	统计公司有多少个不重复的部门
	select count(distinct deptno) from emp;



2）max和min
想法和count一样，所以他们也是多行函数啊。

	范例1：
	查询员工表中最高工资，最低工资
	select max(sal) "最高工资",min(sal) "最低工资"
	from emp;
	
	入职最早，入职最晚员工
	select max(hiredate) "最晚入职时间",min(hiredate) "最早入职时间"
	from emp;

3）sum和avg

	范例1：
	按部门求出该部门平均工资，且平均工资取整数，采用截断
	select deptno "部门编号",trunc(avg(sal),0) "部门平均工资"
	from emp
	group by deptno;
	
	(继续)查询部门平均工资大于2000元的部门
	select deptno "部门编号",trunc(avg(sal),0) "部门平均工资"
	from emp
	group by deptno
	having trunc(avg(sal),0) > 2000; 

## 单引号与双引号出现的地方 ##
	单引号出现的地方如下：
	1）字符串，例如：'hello'
	2）日期型，例如：'17-12月-80'
	3）to_char/to_date(日期,'YYYY-MM-DD HH24:MI:SS')
	
	双引号出现的地方如下：
	1）列别名，例如：select ename "姓 名" from emp
	2）to_char/to_date(日期,'YYYY"年"MM"月"DD"日" HH24:MI:SS')‘’号中的英文字符大小写不敏感

