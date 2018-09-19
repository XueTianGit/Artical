title: Oracle-增、删、改和事务
date: 2016/4/29 8:45:55  
categories: Database
---

# Oracle-增、删、改和事务 #

## 增删改数据 ##

1）增

A 普通的插入数据是遵循SQL99语法的。
例如：
向emp表中插入一条记录 
a， insert into emp values(1111,'JACK','IT',7788,sysdate,1000,100,40);   //按表默认结构顺序
b， insert into emp(ENAME,EMPNO,JOB,MGR,HIREDATE,SAL,COMM,DEPTNO) 
	values('MARRY',2222,'IT',7788,sysdate,1000,100,40);      //自定顺序进行插入
c， insert into emp values(3333,'SISI','IT',7788,sysdate,1000,NULL,40);  //向emp表中插入NULL值， 显示插入； 当然如果我们在插入时没查对应字段，默认就为null值呗。
	NT：前提是所插入的字段允许插入NULL值

B 批量插入
例如：
将xxx_emp表中所有20号部门的员工，复制到emp表中，批量插入，insert into 表名 select ...语法
insert into emp
select * 
from xxx_emp
where deptno=20;
NT：在插入是我们当然可以指定插入的字段

C：在sqlplus工具下，动态插入
动态插入依赖于“&”， &可以运用在任何一个DML语句中。并且应注意，&是sqlplus工具提供的占位符，如果是字符串或日期型要加''符，数值型无需加''符

例如：
a， insert into emp values(&empno,'&ename','&job',&mgr,&hiredate,&sal,&comm,&xxxxxxxx);
b， select * from &table;
c， select empno,ename,&colname from emp;
d， select * from emp where sal > &money;
e， select deptno,avg(sal)
	from emp
	group by &deptno
	having avg(sal) > &money;

2）删
删除数据并没有什么特别。
例如：
a， delete from emp;    //删除emp表中的所有记录
b， 删除工资比所有部门平均工资都低的员工。  ->  这是一个条件未知的事物，优先考虑子查询
    delete 
    from emp 
    where sal < all(
	 	select avg(sal) 
         from emp 
         group by deptno
      ); 
c,  delete from emp where comm is null;  //删除无佣金的员工

3）改
更改数据并没什么特别。
例如：	
a， 将'SMITH'的工资增加20%
	update emp set sal=sal*1.2 where ename = upper('smith');
	
b， 将'SMITH'的工资设置为20号部门的平均工资，这是一个条件未知的事物，优先考虑子查询
     update emp 
     set sal = (
		select avg(sal) 
        from emp 
        where deptno=20	
     )；

## 事务 ##

### 回顾 ###

1）jdbc编程中，如何使用事务？
	connection.setAutoCommit(false);
	pstmt.executeUpdate();   //preparestatement
	connection.commit();
	connection.rollback();

2）hibernate编程中，如何使用事务？
	transaction.begin();
	session.save(new User());
	transaction.commit();
	transaction.rollback();

3）spring编程中，如何使用事务？
	spring可以分为二种
	>编程式事务，藕合
	>声明式事务，解藕，提倡

4）在MySQL中事务的开始是以 start transaction 为开始的。

### Oracle事务 ###
事务开始： 在Oracle中事务的开始是以第一条DML操作做为事务开始。（CRUD）

事务提交：
（1）显示提交：commit	
（2）隐藏提交：DDL/DCL/exit(sqlplus工具)
提交的什么地方呢？ ->  提交是的从事务开始到事务提交中间的内容，提交到ORCL数据库中的DBF二进制文件

事务回滚：
（1）显示回滚：rollback
（2）隐藏回滚：关闭窗口(sqlplus工具)，死机，掉电
注意：回滚到事务开始的地方

回滚点的回顾：
回滚点是在操作之间设置的一个标志位，用于将来回滚之用，如果没有设置回滚点的话，Oracle必须回滚到事务开始的地方，其间做的一个正确的操作也将撤销。
设置回滚点： savepoint a;
回滚到回滚点： rollback to savepoint a;
**Oracle事务回滚实现依赖的机制是实例池 **

#### Oracle事务的隔离级别 ####
Oracle支持：read committed 和 serializable。  默认为read committed

Oracle中设置事务隔离级别为serializable
set transaction isolation level serializable;



**本文摘抄自传智博客笔记。**

