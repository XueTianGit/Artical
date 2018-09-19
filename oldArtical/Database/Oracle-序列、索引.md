title: Oracle-序列、索引
date: 2016/4/28 8:45:55  
categories: Database
---

# Oracle-序列、索引 #

## 序列 ##
	什么是序列【Sequence】
	（1）类似于MySQL中的auto_increment自动增长机制，但Oracle中无auto_increment机制
	（2）是oracle提供的一个产生唯一数值型值的机制
	（3）通常用于表的主健值
	（4）序列只能保证唯一，不能保证连续  
	（5）序列值，可放于内存，取之较快
	
	为什么要用序列
	（1）以前我们为主健设置值，需要人工设置值，容易出错
	（2）以前每张表的主健值，是独立的，不能共享

### 创建与删除序列 ###
	创建语法：
	CREATE SEQUENCE sequence
	       [INCREMENT BY 1]
	       [START WITH 1]
	       [{MAXVALUE  | NOMAXVALUE}]
	       [{MINVALUE n | NOMINVALUE}]
	       [{CYCLE | NOCYCLE}]
	       [{CACHE  | NOCACHE 20 }];
	
	
	例如：
	CREATE SEQUENCE dept_deptid_seq
	                INCREMENT BY 10        //步长为10
	                START WITH 120         //起始值为120
	                MAXVALUE 9999          //最大9998
	                NOCACHE                //不放入缓存
	                NOCYCLE;               ///不循环
	
	
	删除序列
	drop sequence dept_deptid_seq;
	
	修改修改dept_deptid_seq序列的的increment by属性为5
	alter sequence dept_deptid_seq
	increment by 5;

**对于序列其他属性的修改，只要不影响先前使用序列的数据， 那就是可以的。**


### 序列的使用 ###

	查询dept_deptid_seq序列的当前值currval和下一个值nextval
	select emp_empno_seq.currval from dual;   
	select emp_empno_seq.nextval from dual;
	
	在表的字段上使用序列：（empno字段使用序列值）
	insert into emp(empno) values(emp_empno_seq.nextval);     
	insert into emp(empno) values(emp_empno_seq.nextval);
	insert into emp(empno) values(emp_empno_seq.nextval);
	
	empno的可能值为： 9， 10， 11


**有了序列以后，我们还是可以手工的去设置值的**
例如:insert into emp(empno) values(12);


## 索引 Index ##
> 它是是一种快速查询表中内容的机制，类似于新华字典的目录， 运用在表中某个/些字段上，但存储时，独立于表之外。并且索引的存在主要是优化查询的。

为什么要用索引

- 通过指针加速Oracle服务器的查询速度
- 通过rowid快速定位数据的方法，减少磁盘I/O     rowid是oracle中唯一确定每张表不同记录的唯一身份证

索引的特点

	（1）索引一旦建立, Oracle管理系统会对其进行自动维护, 而且由Oracle管理系统决定何时使用索引
	（2）用户不用在查询语句中指定使用哪个索引
	（3）在定义primary key或unique约束后系统自动在相应的列上创建索引
	（4）用户也能按自己的需求，对指定单个字段或多个字段，添加索引

### rowid ###
它的特点：

	（1）位于每个表中，但表面上看不见，例如：desc emp是看不见的
	（2）只有在select中，显示写出rowid，方可看见
	（3）它与每个表绑定在一起，表亡，该表的rowid亡，**二张表rownum可以相同，但rowid必须是唯一的**
	（4）rowid是18位大小写加数字混杂体，唯一表代该条记录在DBF文件中的位置
	（5）rowid可以参与=/like比较时，用''单引号将rowid的值包起来，且区分大小写
	（6）rowid是联系表与DBF文件的桥梁

索引与rowid的关系：
![](http://7xrbxa.com1.z0.glb.clouddn.com/Database-Oracle-%E5%BA%8F%E5%88%97%E3%80%81%E7%B4%A2%E5%BC%951.JPG)

### 创建与删除索引 ###
首先我们应先明白：

	什么时候【要】创建索引
	（1）表经常进行 SELECT 操作
	（2）表很大(记录超多)，记录内容分布范围很广
	（3）列名经常在 WHERE 子句或连接条件中出现
	 注意：符合上述某一条要求，都可创建索引，创建索引是一个优化问题，同样也是一个策略问题
	       
	什么时候【不要】创建索引
	（1）表经常进行 INSERT/UPDATE/DELETE 操作
	（2）表很小(记录超少)
	（3）列名不经常作为连接条件或出现在 WHERE 子句中


	语法：create index 索引名 on 表名(字段,...)
	
	例如：
	为emp表的empno单个字段，创建索引emp_empno_idx，  ->  单列索引，
	create index emp_empno_idx
	on emp(empno);
	
	为emp表的ename,job多个字段，创建索引emp_ename_job_idx，    ->  多列索引/联合索引
	create index emp_ename_job 
	on emp(ename,job);

> 应注意：
> 如果在where中只出现job不使用索引
> 如果在where中只出现ename使用索引
> 我们提倡同时出现ename和job

**注意：索引创建后，只有查询表有关，和其它（insert/update/delete）无关,解决速度问题**


	删除emp_empno_idx和emp_ename_job_idx索引，drop index 索引名
	drop index emp_empno_idx;
	drop index emp_ename_job_idx;






  