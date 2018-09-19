title: JavaWebFoundation-事务与JDBC
date: 2016/3/1 9:18:40               
categories: JavaWebFoundation
---

# 事务与JDBC #

## 什么是事务 ##
> 事务是指逻辑上的一组操作，组成这组操作的各个单元要不全部成功，要不全部失败！

事务四大特性 (ACID) 

* 原子性(Atomicity)：事务中的全部操作在数据库中是不可分割的，要么全部完成，要么均不执行。

* 一致性(Consistency)：几个并行执行的事务，其执行结果必须与按某一顺序串行执行的结果相一致。

* 隔离性(Isolation)：事务的执行不受其他事务的干扰，事务执行的中间结果对其他事务必须是透明的。

* 持久性(Durability)：对于任意已提交事务，系统必须保证该事务对数据库的改变不被丢失，即使数据库出现故障。 

> 不考虑隔离性会引发的问题：

> * 脏读：一个事务读取到另一个事务未提交的数据
> * 不可重复读：在一个事务内读取表中某一行数据，多次读取的结果不同（一个事务读取到另一个事务提交的数据）
> 例如：一个编辑人员两次读取同一文档，但在两次读取之间，作者重写了该文档。当编辑人员第二次读取文档时，文档已更改。原始读取不可重复。
> 如果只有在作者全部完成编写后编辑人员才可以读取文档，则可以避免该问题。
> 
> * 虚读（幻读）：是指在一个事务内读取到别的事务插入的数据，导致前后读取不一致。（涉及到多条记录， 数量不一致了）

	
## 数据库的四种隔离级别 ##
> 不同的隔离级别，可以避免不同的问题

- Serializable：避免所有问题
- Repeatable read:可避免脏读、不可重复读的情况发生
- Read conmitted：可以避免脏读
- Read uncommitted：什么都不能避免
> 
	设置数据库事务的隔离级别（mysql）: set transaction isolation level xxx
	查询当前事务的隔离级别： select  @@tx_isolation 
	
	MySQL支持全部隔离级别， 且默认为 Repeatable read
	Oracle只支持 Serializable和Read conmitted，且默认为Read conmitted

## JDBC与事务 ##
### 使用事务 ###
> NT： 当JDBC程序向数据库获得一个Connection对象时，默认情况下这个Connection对象会自动向数据库提交在它上面发送的sql语句。

	conn.setAutoCommit(false); // 开启事务
	conn.rollBack(); // 回滚事务
	conn.commit(); //提交事务

	事务回滚点：
	数据库默认会回滚到原点， 不过我们一个科设置事务的回滚点。建回滚点前的sql语句会执行，不过在回滚后一定要记得提交事务.
	example:
	
		/*....*/
		sp = conn.setSavePoint();
		/*...*/
	
		finally{
			conn.rollback(sp);
			conn.commit();
		}