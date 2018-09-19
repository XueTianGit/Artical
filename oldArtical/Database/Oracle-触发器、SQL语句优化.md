title: Oracle-触发器、SQL语句优化
date: 2016/4/12 8:45:55  
categories: Database
---

# Oracle-触发器、SQL语句优化 #

## 触发器 ##

	不同的DML（CRUD）操作，触发器能够进行一定的拦截，符合条件的操作方可操作基表，反之不可操作基表。
	类似于Filter、Interceptor。
	
	为什么要使用触发器呢？
	对DML操作做限制，防止其限制的操作基表。
	
	触发器的类型分为：
	语句级触发器和行级触发器。

### 创建触发器 ###
	语法：
	CREATE  [or REPLACE] TRIGGER  触发器名
	{BEFORE | AFTER}
	{ INSERT | DELETE| UPDATE OF 列名}
	ON  表名
	[FOR EACH ROW]
	PLSQL 块【declare…begin…end;/】
	
	删除触发器：
	drop trigger 触发器名


#### 语句级触发器 ####
	语句级触发器：  在指定的操作语句操作之前或之后执行一次，不管这条语句影响了多少行 。
	
	范例：创建语句级触发器deleteEmpTrigger，当对表【emp】进行删除【delete】操作后【after】，显示"hello world"
	create trigger deleteEmpTrigger 
	before delete on emp
	begin
		dbms_output.put_line('hello world');
	end;
	/

> 关与hello world的打印：
> 对于deleteEmpTrigger，我们在删除一条记录时，hello world会打印一次。
> 当我们在一次删除多条记录时，hello world还是只会打印一次


#### 行级触发器 ####
触发语句作用的每一条记录都被触发。在行级触发器中使用:old和:new伪记录变量, 识别值的状态。

	范例：
	创建行级触发器checkSalaryTrigger，涨后工资这一列，确保大于涨前工资。
	
	create trigger checkSalaryTrigger 
	after update of sal on emp
	for each row
	begin
		if :new < :old then
			raise_application_error("-20200", "你到底是涨工资还是降工资？");
		end if;
	end;
	/
**在该例中，如果更新sal列的值不满足条件是操作是不会成功的！！！**


> 对于after的理解：
> 从这个例子可以看出， 触发器和Filter和Interceptor还是不同的。
> 不同点：Filter和Interceptor的“后”指的是，能让已经更改完毕，或是业务已经完毕后再执行内容。对于触发器， 它的after相当于是检测对表的修改是否符合要求， 如果不符合要求，它是不会让这个操作执行成功的， 即他不是在影响了表的内容之后再执行的意思。


### 触发器的一些问题 ###
	1）删除触发器后，表肯定还是存在的
	2）对于表的删除，只要是不是完全删除（drop table purge），即表还在回收站中，那么触发器还是存在的。
	   表闪回后还是可以继续使用的。 


## Oracle SQL语句执行的优化 ##
	随着实际项目的启动，Oracle经过一段时间的运行，最初的Oracle设置，会与实际Oracle运行性能会有一些差异，这时我们就需要做一个优化调整。
	Oracle可分为四大类：
	       》主机性能
	       》内存使用性能
	       》网络传输性能
	       》SQL语句执行性能【程序员】
	
	这里我还只能关注SQL语句执行性能优化这个课题。

**以下优化大部分优化方案的核心是： 尽量不要让Oracle直接去做CRUD， 而一些CRUD的前奏（比如计算啦）我们应帮他完成。**

优化方案：

	01）选择最有效率的表名顺序
	ORACLE的解析器按照从右到左的顺序处理FROM子句中的表名， 
	FROM子句中写在最后的表将被最先处理，
	在FROM子句中包含多个表的情况下,你必须选择记录条数最少的表放在最后，
	如果有3个以上的表连接查询,那就需要选择那个被其他表所引用的表放在最后。
	例如：查询员工的编号，姓名，工资，工资等级，部门名
		select emp.empno,emp.ename,emp.sal,salgrade.grade,dept.dname
		from salgrade,dept,emp
		where (emp.deptno = dept.deptno) and (emp.sal between salgrade.losal and salgrade.hisal)  		
	1)**如果三个表是完全无关系的话，将记录和列名最少的表，写在最后，然后依次类推**
	2)**如果三个表是有关系的话，将引用最多的表，放在最后，然后依次类推**
	
	
	（02）WHERE子句中的连接顺序
	ORACLE采用自右而左的顺序解析WHERE子句,根据这个原理,表之间的连接必须写在其他WHERE条件之左,
	**那些可以过滤掉最大数量记录的条件必须写在WHERE子句的之右。**  
	例如：查询员工的编号，姓名，工资，部门名  
		select emp.empno,emp.ename,emp.sal,dept.dname
		from emp,dept
		where (emp.deptno = dept.deptno) and (emp.sal > 1500)   
	  	
	（03）SELECT子句中避免使用*号
	ORACLE在解析的过程中,会将*依次转换成所有的列名，这个工作是通过查询数据字典完成的，这意味着将耗费更多的时间
	select empno,ename from emp;
	
	（04）使用DECODE函数来减少处理时间
	使用DECODE函数可以避免重复扫描相同记录或重复连接相同的表
	
	（05）整合简单，无关联的数据库访问
	
	（06）用TRUNCATE替代DELETE
	   
	（07）尽量多使用COMMIT； 这是因为COMMIT会释放回滚点
	
	（08）用WHERE子句替换HAVING子句（即条件的过滤，能越提前，就提前）-> WHERE先执行，HAVING后执行
	     
	（09）多使用内部函数提高SQL效率
	     
	（10）使用表的别名
	     
	（11）使用列的别名
	
	（12）用索引提高效率
	      
	（13）字符串型，能用=号，不用like  
	
	（14）SQL语句用大写的; 因为Oracle服务器总是先将小写字母转成大写后，才执行
	     
	（15）**避免在索引列上使用NOT**; 因为Oracle服务器遇到NOT后，他就会停止目前的工作，转而执行全表扫描
	
	（16）**避免在索引列上使用计算**
	  WHERE子句中，如果索引列是函数的一部分，优化器将不使用索引而使用全表扫描，这样会变得变慢 
	  例如，SAL列上有索引，
		  低效：
		  SELECT EMPNO,ENAME
		  FROM EMP 
		  WHERE SAL*12 > 24000;
		  高效：
		  SELECT EMPNO,ENAME
		  FROM EMP
		  WHERE SAL > 24000/12;
	
	（17）用 >= 替代 >
	  低效：
	  SELECT * FROM EMP WHERE DEPTNO > 3   
	  首先定位到DEPTNO=3的记录并且扫描到第一个DEPT大于3的记录
	  高效：
	  SELECT * FROM EMP WHERE DEPTNO >= 4  
	  直接跳到第一个DEPT等于4的记录
	
	（18）用IN替代OR
	
	（19）总是使用索引的第一个列
	      如果索引是建立在多个列上，只有在它的第一个列被WHERE子句引用时，优化器才会选择使用该索引
	      当只引用索引的第二个列时，不引用索引的第一个列时，优化器使用了全表扫描而忽略了索引
	      create index emp_sal_job_idex
	      on emp(sal,job);
	      ----------------------------------
	      select *
	      from emp  
	      where job != 'SALES';	      
	
	（20）避免改变索引列的类型，显示比隐式更安全 
	 当字符和数值比较时，ORACLE会优先转换数值类型到字符类型 
	 select 123 || '123' from dual;
      
	







