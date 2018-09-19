title: Oracle-视图、同义词
date: 2016/4/17 8:45:55  
categories: Database
---

# Oracle-视图、同义词 #


## 视图 ##

	什么是视图【View】 
	（1）视图是一种虚表 
	（2）视图建立在已有表的基础上, 视图赖以建立的这些表称为基表
	（3）向视图提供数据内容的语句为 SELECT 语句,可以将视图理解为存储起来的 SELECT 语句
	（4）视图向用户提供基表数据的另一种表现形式
	（5）视图没有存储真正的数据，真正的数据还是存储在基表中
	（6）程序员虽然操作的是视图，但最终视图还会转成操作基表
	（7）一个基表可以有0个或多个视图 

什么时候去用视图？

	（1）如果你不想让用户看到所有数据（字段，记录），只想让用户看到某些的数据时，此时可以使用视图
	（2）当你需要减化SQL查询语句的编写时，可以使用视图，但不提高查询效率
	
	视图应用领域->  银行，电信，金属，证券军事等不便让用户知道所有数据的项目中

视图的作用：

	（1）限制数据访问
	（2）简化复杂查询
	（3）提供数据的相互独立
	（4）同样的数据，可以有不同的显示方式


###  创建视图 ###
视图肯定是基于某个表来创建的，包装的是查询语句。

	例如：
		create view emp_view_1
		as
		select * from emp;
	
	也可以这样创建：
		create or replace view emp_view_1
		as
		select * from emp;
	->  该视图的字段名称即为emp表的字段名称。

以上创建的两个视图并不好， 我们使用视图，主要是用来查询的所有，应创建只能用来查询的视图

	create view emp_view_1
	as
	select * from emp
	with read only;


NT：默认情况下，普通用户无权创建视图，得让sys用户为你分配creare view的权限 

	以sysdba身份，授权scott用户create view权限
	grant create view to scott;
	
	以sysdba身份，撤销scott用户create view权限
	revoke create view from scott;

一些创建视图的范例：

	a， 基于emp表指定列，创建视图emp_view_2，该视图包含编号/姓名/工资/年薪/年收入     注意 ： **查询中使用列别名**
		create view emp_view_2
		as
		select empno "编号",ename "姓名",sal "工资",sal*12 "年薪",sal*12+NVL(comm,0) "年收入"
		from emp;
	b， 基于emp表指定列，创建视图emp_view_3(a,b,c,d,e)，包含编号/姓名/工资/年薪/年收入（视图中使用列名）
		create view emp_view_3(a,b,c,d,e)
		as
		select empno "编号",ename "姓名",sal "工资",sal*12 "年薪",sal*12+NVL(comm,0) "年收入"
		from emp;

> a与b两个范例有什么区别呢？ 区别在于我们查看创建的视图的字段的名称。
> 在我们创建视图时如果不指定字段的名称， 则名称即为查询字段的名称， 当指定是，就以指定的为准

	c 创建视图emp_view_4，视图中包含各部门的最低工资，最高工资，平均工资
		create or replace view emp_view_4
		as
		select deptno "部门号",min(sal) "最低工资",max(sal) "最高工资",round(avg(sal),0) "平均工资"
		from emp
		group by deptno;

### 视图的删除与修改操作 ###

	1）使用delete去删除视图中一条记录会影响基表的内容， 即基表的数据也被删除
	例如：
		delete from emp_view_1 where empno=7788;
	2）删除整个视图，并不会影响基表

**当然，照我们所想，对视图的增删改操作应该都得是不可以的。为了保证这样我们在创建视图时应该使用with read only**

	create or replace view emp_view_1
	as
	select * from emp
	with read only;   // 这样就只能利用视图来查询了。

还应注意： ** 删除视图，不会进入回收站**


## 同义词 ##
同义词就是对一些比较长名字的对象(表，视图，索引，序列，。。。）做减化，用别名替代。  **就是别名**

	同义词的作用：
	（1）缩短对象名字的长度
	（2）方便访问其它用户的对象

### 创建同义词 ###
	create synonym 同义词 for 表名/视图/其它对象
	例如：
	创建与salgrade表对应的同义词。
	create synonym e for salgrade;
	create synonym ev5 for emp_view_5;
	
	注意的是，普通用户在没有权限的情况下，是不能创建同义词的。
		以sys身份授予scott普通用户create synonym权限
		grant create synonym to scott;
		
		以sys身份从scott普通用户撤销create synonym权限
		revoke create synonym from scott;
	
	**同义词创建以后，我们就可以使用同义词操作其代表的事物了。**
	
	删除同义词
	drop synonym ev5;
	
	**删除同义词，不会影响基表。而删除基表当然会影响同义词**


**本文摘抄自传智博客笔记**