title: Oracle-分页与集合查询
date: 2016/4/15 8:45:55  
categories: Database
---

# Oracle-分页与集合查询 #

## 分页 ##
> 在MySQL中分页使用的是limit关键字。
> 
	例如:select * from user limit 0, 5;   ->  0代表索引号，是第一条数据； 5代表是自索引号开始的5条数据。
		即查询出user表的前5条数据。
	而在hibernate中进行分页就更简单了，根本无需我们关心底层数据库的结果，直接两个API搞定。
例如：

	Query.setFirstResult(0);     类似mysql的索引号   
	Query.setMaxResult(5); 	     查询最多5条， 若不足，有几条查出几条

### rownum ###
想要理解oracle分页，首先应知道oracle中的rownum关键字。
那么什么是rownum呢？

	1）rownum是oracle专用的关健字
	2）rownum与表在一起，表亡它亡,表在它在 
	3）rownum在默认情况下，从表中是查不出来的  
	4）只有在select子句中，明确写出rownum才能显示出来  ->  select rownum, emp.* from emp;  这样就可以查出
	5）rownum是number类型，且唯一连续              
	6）rownum最小值是1，最大值与你的记录条数相同
	7）rownum也能参与关系运算
	   * rownum = 1    有值
	   * rownum < 5    有值	
	   * rownum <=5    有值 		
	   * rownum > 2    无值    	
	   * rownum >=2    无值
	   * rownum <>2    有值	与  rownum < 2 相同
	   * rownum = 2    无值
	8）**基于rownum的特性，我们通常rownum只用于<或<=关系运算 **  

范例： 利用rownum，显示emp表中3-8条记录

	select rownum "伪列",emp.* from emp where rownum<=8      （8条记录）
	minus                                                   （取差集）
	select rownum,emp.* from emp where rownum<=2;           （2条记录）

**这个例子给出在oracle中解决分页的方案， 仅仅这样是不行的，oracle中的分页还依赖与子查询**

### 结合子查询完成oracle分页 ###
标准范例：

	显示emp表中3-8条记录
	select xx.*
	from (select rownum ids,emp.* from emp where rownum<=8) xx 
	where ids>=2;


**可以看出， oracle分页的实现主要时利用rownum字段和子查询完成**


## 集合查询 ##

想要弄明白集合查询，其实很简单，只要明白数学中的并集、交集、差集就差不多了。

### union ###
使用union关键字， 我们可以取两次查询结果的并集。
例如： 查询20号部门或30号部门的员工信息

	select * from emp where deptno = 20
	union
	select * from emp where deptno = 30;

### union all ###
	union all 与union只有一点不同：
	union：二个集合中，如果都有相同的，取其一
	union all：二个集合中，如果都有相同的，都取

### intersect ###
> 使用intersect关键字， 我们可以取两次查询结果的交集。

例如：查询工资在1000-2000和1500-2500之间的员工信息

	select * from emp where sal between 1000 and 2000
	intersect
	select * from emp where sal between 1500 and 2500;
	
	方式2：
	select * 
	from emp
	where (sal between 1000 and 2000) and (sal between 1500 and 2500);

### minus ###
> 使用minust关键字， 我们可以取两次查询结果的差集。

例如：查询工资在1000-2000，但不在1500-2500之间的员工信息

	select * from emp where sal between 1000 and 2000
	minus
	select * from emp where sal between 1500 and 2500;

	方式2：
	select * 
	from emp 
	where (sal between 1000 and 2000) and (sal not between 1500 and 2500);

## 集合查询的细节 ##
	1）集合操作时，必须确保集合列数是相等
	
	例如：
	
		select empno,ename,sal,comm from emp where deptno = 20
		union
		select empno,ename,sal from emp where deptno = 30;      错
	
	2）集合操作时，必须确保集合列类型对应相同
	3）A union B union C = C union B union A
	4）当多个集合操作时，结果的列名由第一个集合列名决定


**从上面也可已看出，集合查询能够完成的工作，使用其他方式也是可以完成的， 一般集合查询最后考虑**

本文中所用到的表：
![](C:\Users\Administrator\Desktop\TheLastTaskOf\博客的html文件\Database\图片\emp表.jpg)