title: Oracle-oracleSQL对单表各种查询操作及函数的使用
date: 2016/4/10 8:45:55  
categories: Database
---



# Oracle-oracleSQL对单表各种查询操作及函数的使用 #

> 安装完成Oracle数据库之后， Oracle本身就带了一些表，例如ORCL数据库中的emp表，这里以emp表为例。使用sqlplus客户端工具进行操作。



## SQLPLUS命令的特点 
	1）是oracle自带的一款工具，在该工具中执行的命令叫SQLPLUS命令
	2）SQLPLUS工具的命令中的关健字可以简写，也可以不简写，例如：col ename for a10;
	3）大小写不敏感，提倡大写
	4）不能够对表数据进行增删改查操作，只能完成显示格式控制，例如：设置显示列宽，清屏，记录执行结果
	5）可以不用分号结束，也可以用分号结束，个人提倡不管SQL或SQLPLUS，都以分号结束
	6）通常称做命令，是SQLPLUS工具中的命令
	   注意：SQLPLUS命令是SQLPLUS工具中特有的语句


## 常用的一些命令 ##

	exit        退出sqlplus工具    
	show user   查询当前用户是谁 
	/           执行最近一次sql语句
	host cls    清屏，属于SQL*PLUS工具中的命令
	desc emp    查询表（emp）的结构


## sqlplus下设置表显示格式 ##
> 在使用sqlplus查看数据库数据时，可能数据的显示杂乱无章。我们可以把他设置的显示更美观一些。

	1）设置显示的列宽（字符型varchar2、日期型date），10个宽度位，a表示字符型，大小写均可
	   column ename format a12;
	   column hiredate format a10;   --12个宽度位--
	
	2）设置显示的列宽（数值型number），9表示数字型，一个9表示一个数字位，四个9表示四个数字位，只能用9
	   column empno format 9999;
	   column mgr format 9999;
	   column sal format 9999;
	   column comm format 9999;
	   column deptno format 9999;
	3）设置一页显示80个条记录的高度
	   set pagesize 80;

## select语句实例 ##

	查询emp表的所有内容，  *号表示通配符，表示该表中的所有字段，但*号不能和具体字段一起使用
	select * from emp;
	或
	select empno,ename,sal,deptno from emp;
	
	查询emp表的员工编号，姓名，工资，部门号，列名，大小写不敏感，但提倡大写
	select empno "编号",ename "姓名",sal "工资",deptNO "部门号" FROM Emp;
	
	**查询emp表的不重复的工作**
	select distinct job from emp;
	
	查询员工的编号，姓名，月薪，年薪（月薪*12)
	select empno,ename,sal,sal*12 "年薪" from emp;
	
	查询员工的编号，姓名，入职时间，月薪，年薪，年收入（年薪+奖金)
	select empno "编号",ename"姓名",hiredate "入职时间",sal "月薪",sal*12 "年薪",sal*12+comm "年收入" from emp;

### 解决查询结果为null引起的问题 ###

	由于null与具体数字进行运算仍为null， 而在在sqlplus客户端工具中，是不显示null这个值的。
	
	解决null的问题，使用NVL()函数，NVL(a,b)：如果a是NULL，用b替代;如果a是非NULL，就不用b替代，直接返回a的值\
	select NVL(null,10) from emp;   结果有14行记录
	select NVL(null,10) from dual;   结果有1行记录
	
	例如：
	select empno "编号",ename"姓名",hiredate "入职时间",sal "月薪",sal*12 "年薪",sal*12+NVL(comm,0) "年收入" 
	from emp;   （在这里，如果comm为null， 则就用0代替）
	
	
	
	使用列别名，查询员工的编号，姓名，月薪，年薪，年收入（年薪+奖金)，AS大小写都可且可以省略AS，**别名用双引号**
	select empno AS "编号",ename as "姓名",sal "月薪" 
	from emp;
	或
	select empno AS 编号,ename as 姓名,sal 月薪 
	from emp;


->区别

	select empno AS "编号",ename as 姓名,sal "月    薪" 
	from emp;
	不加双引号的别名不能有空格；加了双引号的别名可以有空格
	要加只能加双引号，不能加单引号，因为在oracle中单引号表示字符串类型或者是日期类型

列名不能使用单引号，因为oracle认为单引号是字符串型或日期型


### 字符串连接符号|| ###
	范例：
	1）使用字符串连接符号||，输出"hello world"，**在oracle中from是必须写的**
	   select 'hello' || ' world' "结果" from dual;
	
	2）使用字符串连接符号||，显示如下格式信息：****的薪水是****美元
	   select ename || '的薪水是' || sal || '美元' 
	   from emp; 


NT：dual哑表或者伪表，只有一条数据

### sysdate ###

	sysdate是oracle的关键在， 注意他， 可以用它来显示系统当前时间，在默认情况下，oracle只显示日期，而不显示时间，格式：26-4月-15
	select sysdate from dual;


### spool命令与@ ###

	使用spool命令，保存SQL语句到硬盘文件e:/oracle-day01.sql，并创建sql文件
	spool e:/oracle-day01.sql;
	
	使用spool off命令，保存SQL语句到硬盘文件e:/oracle-day01.sql，并创建sql文件，结束语句
	spool off;
	
	**这样会把在命令窗口中执行在 spool e:/oracle-day01.sql; 和 spool off; 之间的sql语句保存到本地硬盘**
	
	使用@命令，将硬盘文件e:/crm.sql，读到orcl实例中，并执行文件中的sql语句
	@ e:/crm.sql; 

### oracle的注释 ###

	1）单行注释
	使用--符号，设置单行注释
	--select * from emp;
	
	使用/* */符号，设置多行注释
	/*
	select
	*
	from 
	emp;
	*/

### 单引号与双引号 ###
单引号出现的地方如下：

	1）字符串型，例如：'hello' || ' world'
	2）日期型，例如'25-4月-15'
	
	双引号出现的地方如下：
	1）列别名，例如：sal*12 "年 薪"，或 sal*12 年薪，个人提倡用""双引号作列别名
	
	
	
	查询emp表中20号部门的员工信息
	select * from emp where deptno = 20;
	
	查询姓名是SMITH的员工，字符串使用''，内容大小写敏感
	select * from emp where ename = 'SMITH';
	
	查询1980年12月17日入职的员工，注意oracle默认日期格式（DD-MON-RR表示2位的年份)
	select * from emp where hiredate = '17-12月-80';
	
	查询工资大于1500的员工
	select * from emp where sal > 1500;
	
	查询工资不等于1500的员工**【!=或<>】**
	select * from emp where sal <> 1500;
	
	查询薪水在1300到1600之间的员工，包括1300和1600
	select * from emp where (sal>=1300) and (sal<=1600);
	或
	select * from emp where sal between 1300 and 1600;
	
	查询薪水不在1300到1600之间的员工，不包括1300和1600
	select * from emp where sal NOT between 1300 and 1600;
	
	查询入职时间在"1981-2月-20"到"1982-1月-23"之间的员工
	select * from emp where hiredate between '20-2月-81' and '23-1月-82';
	注意：
	1)对于数值型，小数值在前，大数值在后
	2)对于日期型，年长值在前，年小值在后


### in(....) ###

	一般我们进行多个数据的查询时：
	查询20号或30号部门的员工
	select * from emp where (deptno=20) or (deptno=30);
	
	
	使用in(....)
	select * from emp where deptno in (30,20);
	查询不是20号或30号部门的员工
	select * from emp where deptno NOT in (30,20);
	
	查询工资是1500或3000或5000的员工 
	select * 
	from emp 
	where sal in (4000,10000,1500,3,300,3000,5000);   //**！**
	
	select * from emp where deptno in (10,20,30,50,'a');   //错误！


### 模糊查询 ###

	%   表示0个，1个或多个字符
	-    表示一个字符
	
	> 凡是精确查询用=符号
	> 凡是不精确查询用like符号，我们通常叫模糊查询
	
	范例：
	
	查询姓名以大写字母S开头的员工，使用
	select * from emp where ename like 'S';
	等价
	select * from emp where ename = 'S';
	select * from emp where ename like 'S%';
	
	查询姓名以大写字母N结束的员工
	select * from emp where ename like '%N';
	
	查询姓名第一个字母是T，最后一个字母是R的员工
	select * from emp where ename like 'T%R';
	
	查询姓名是4个字符的员工，且第二个字符是I
	select * from emp where ename like '_I__';
	插入一条姓名为'T_IM'的员工，薪水1200
	insert into emp(empno,ename) values(1111,'T_IM');
	
	**查询员工姓名中含有'_'的员工，使用\转义符，让其后的字符回归本来意思【like '%\_%' escape '\'】**
	
	select * from emp where ename like '%\_%' escape '\';   （转义 "\" 后面的字符）
	 
	**插入一个姓名叫'的员工**
	insert into emp(empno,ename) values(2222,'''');   （'' 代表了'）
	
	**插入一个姓名叫''的员工**
	insert into emp(empno,ename) values(2222,'''''');
	
	
	查询所有员工信息，使用%或%%
	select * from emp;
	select * from emp where ename like '%';
	select * from emp where ename like '%_%';


### is 的使用 ###

	查询佣金为null的员工
	select * from emp where comm is null;
	注意：null不能参与=运算
	      null能参与number/date/varchar2类型运算
	
	查询佣金为非null的员工
	select * from emp where comm is not null;
	
	查询无佣金且工资大于1500的员工
	select * 
	from emp 
	where (comm is null) and (sal>1500); 


### not一般都会损失性能  ###

	查询职位是"MANAGER"或职位不是"ANALYST"的员工（方式一，使用!=或<>）
	select *
	from emp
	where (job='MANAGER') or (job<>'ANALYST');
	
	查询职位是"MANAGER"或职位不是"ANALYST"的员工（方式二，使用not）  //not一般都会损失性能
	select *
	from emp
	where (job='MANAGER') or (not(job='ANALYST'));

## order by 子句 ##

	order by后面可以跟列名、别名、表达式、列号（从1开始，在select子句中的列号）
	列名:
	select empno,ename,sal,hiredate,sal*12 "年薪" 
	from emp
	order by hiredate desc;
	
	别名: 
	select empno,ename,sal,hiredate,sal*12 "年薪" 
	from emp
	order by "年薪" desc;
	
	表达式:
	select empno,ename,sal,hiredate,sal*12 "年薪" 
	from emp
	order by sal*12 desc;
	
	列号，从5开始：
	select empno,ename,sal,hiredate,sal*12 "年薪" 
	from emp
	order by 5 desc;



	
	查询员工信息（编号，姓名，月薪，年薪），**按月薪升序排序，默认升序，如果月薪相同，按oracle内置的校验规则排序**
	select empno,ename,sal,sal*12 
	from emp 
	order by sal asc; 
	
	查询员工信息（编号，姓名，月薪，年薪），按月薪降序排序
	select empno,ename,sal,sal*12 
	from emp 
	order by sal desc; 
	
	查询员工信息，按入职日期降序排序，使用列名
	select empno,ename,sal,hiredate,sal*12 "年薪" 
	from emp
	order by hiredate desc;



	查询员工信息，按佣金升序或降序排列，**null值看成最大值**
	select * from emp order by comm desc;
	
	查询员工信息，对有佣金的员工，按佣金降序排列，**当order by 和 where 同时出现时，order by 在最后**
	select *
	from emp
	where comm is not null
	order by comm desc;
	
	查询员工信息，按工资降序排列,相同工资的员工再按入职时间降序排列
	select *
	from emp
	order by sal desc,hiredate desc;
	
	select *
	from emp
	order by sal desc,hiredate asc;
	注意：只有当sal相同的情况下，hiredate排序才有作用
	
	查询20号部门，且工资大于1500，按入职时间降序排列
	select *
	from emp
	where (deptno=20) and (sal>1500)
	order by hiredate desc;


## 单行函数 ##

> 单行函数：只有一个参数输入，只有一个结果输出
> 多行函数或分组函数：可有多个参数输入，只有一个结果输出		

### lower/upper/initca ###
	select lower('www.BAIdu.COM') from dual;   // 全部转换成大写
	select upper('www.BAIdu.COM') from dual;
	select initcap('www.BAIdu.COM') from dual;  //首字母大写

### concat/substr ###

	select concat('hello','你好') from dual;    //正确
	select concat('hello','你好','世界') from dual;    //错误
	select 'hello' || '你好' || '世界' from dual;    //正确
	select concat('hello',concat('你好','世界')) from dual;   //  正确
	select substr('hello你好',5,3) from dual;

> 5表示从第几个字符开始算，第一个字符为1，中英文统一处理
> 3表示连续取几个字符

### length/lengthb ###
	测试length/lengthb函数，编码方式为UTF8/GBK(赵君)，一个中文占3/2个字节长度，一个英文一个字节
	select length('hello你好') from dual; 
	select lengthb('hello你好') from dual; 

### instr/lpad/rpad ###
	测试instr/lpad/rpad函数，从左向右找第一次出现的位置，从1开始
	select instr('helloworld','o') from dual;
	注意：找不到返回0
	      大小写敏感 
	select LPAD('hello',10,'#') from dual;
	select RPAD('hello',10,'#') from dual;

### trim/replace ###
	测试trim/replace函数
	select trim(' ' from '  he  ll                ') from dual;
	select replace('hello','l','L') from dual;

### round/trunc/mod ###
	测试round/trunc/mod函数作用于数值型
	select round(3.1415,3) from dual;   //4舍五入
	select trunc(3.1415,3) from dual;   //按精度截取
	select mod(10,3) from dual;   //取余


#### round作用于日期 ####

	sysdate = 26-4月-15
	
	测试round作用于日期型（month）
	select round(sysdate,'month') from dual;   ->1-5月-15 
	
	测试round作用于日期型（year）
	select round(sysdate,'year') from dual;    ->1-1月-14
	
	显示昨天，今天，明天的日期，日期类型 +- 数值 = 日期类型
	select sysdate-1 "昨天",sysdate "今天",sysdate+1 "明天" from dual;


#### months_between ####
	使用months_between函数，精确计算到年底还有多少个月
	select months_between('31-12月-15',sysdate) from dual;
	
	使用months_between函数，以精确月形式显示员工工龄
	select ename "姓名",months_between(sysdate,hiredate) "精确月工龄" from emp;

#### add_months ####

	测试add_months函数，下个月今天是多少号
	select add_months(sysdate,1) from dual;
	
	测试add_months函数，上个月今天是多少号
	select add_months(sysdate,-1) from dual;


#### next_day ####
	测试next_day函数，从今天开始算，下一个星期三是多少号【中文平台】
	select next_day(sysdate,'星期三') from dual;
	
	测试next_day函数，从今天开始算，下下一个星期三是多少号【中文平台】
	select next_day(next_day(sysdate,'星期三'),'星期三') from dual;
	
	测试next_day函数，从今天开始算，下一个星期三的下一个星期日是多少号【中文平台】
	select next_day(next_day(sysdate,'星期三'),'星期日') from dual;


#### last_day ####
	测试last_day函数，本月最后一天是多少号
	select last_day(sysdate) from dual;
	
	测试last_day函数，本月倒数第二天是多少号
	select last_day(sysdate)-1 from dual;
	
	测试last_day函数，下一个月最后一天是多少号
	select last_day(add_months(sysdate,1)) from dual;
	
	测试last_day函数，上一个月最后一天是多少号
	select last_day(add_months(sysdate,-1)) from dual;
	
	注意：
	1）日期-日期=天数
	2）日期+-天数=日期


## oracle中三大类型与隐式数据类型转换 ##

	一些隐式转换：
	（1）varchar2变长/char定长-->number，例如：'123'->123
	（2）varchar2/char-->date，例如：'25-4月-15'->'25-4月-15'
	（3）number---->varchar2/char，例如：123->'123'
	（4）date------>varchar2/char，例如：'25-4月-15'->'25-4月-15'
	
	> oracle如何隐式转换：
	> 1）=号二边的类型是否相同
	> 2）如果=号二边的类型不同，尝试的去做转换
	> 3）在转换时，要确保合法合理，否则转换会失败，例如：12月不会有32天，一年中不会有13月
 


### 显示类型转换 ###

![](http://7xrbxa.com1.z0.glb.clouddn.com/Database-Oracle-oracleSQL%E5%AF%B9%E5%8D%95%E8%A1%A8%E5%90%84%E7%A7%8D%E6%9F%A5%E8%AF%A2%E6%93%8D%E4%BD%9C%E5%8F%8A%E5%87%BD%E6%95%B0%E7%9A%84%E4%BD%BF%E7%94%A81.png)

	查询1980年12月17日入职的员工（方式一：日期隐示式转换）
	select * from emp where hiredate = '17-12月-80';
	
	使用to_char(日期，'格"常量"式')函数将日期转成字符串，显示如下格式：2015 年 04 月 25 日 星期六
	select to_char(sysdate,'yyyy" 年 "mm" 月 "dd" 日 "day') from dual;
	
	使用to_char(日期，'格式')函数将日期转成字符串，显示如格式：2015-04-25今天是星期六 15:15:15
	select to_char(sysdate,'yyyy-mm-dd"今天是"day hh24:mi:ss') from dual;
	或
	select to_char(sysdate,'yyyy-mm-dd"今天是"day HH12:MI:SS AM') from dual;
	
	使用to_char(数值，'格式')函数将数值转成字符串，显示如下格式：$1,234
	select to_char(1234,'$9,999') from dual;
	
	使用to_char(数值，'格式')函数将数值转成字符串，显示如下格式：￥1,234select to_char(1234,'$9,999') from dual;
	select to_char(1234,'L9,999') from dual;
	
	使用to_date('字符串','格式')函数，查询1980年12月17日入职的员工（方式二：日期显式转换）
	select * from emp where hiredate = to_date('1980年12月17日','yyyy"年"mm"月"dd"日"');
	或
	select * from emp where hiredate = to_date('1980#12#17','yyyy"#"mm"#"dd');
	或
	select * from emp where hiredate = to_date('1980-12-17','yyyy-mm-dd');
	
	使用to_number('字符串')函数将字符串‘123’转成数字123
	select to_number('123') from dual;


格式：

![](http://7xrbxa.com1.z0.glb.clouddn.com/Database-Oracle-oracleSQL%E5%AF%B9%E5%8D%95%E8%A1%A8%E5%90%84%E7%A7%8D%E6%9F%A5%E8%AF%A2%E6%93%8D%E4%BD%9C%E5%8F%8A%E5%87%BD%E6%95%B0%E7%9A%84%E4%BD%BF%E7%94%A81.png)

![](http://7xrbxa.com1.z0.glb.clouddn.com/Database-Oracle-oracleSQL%E5%AF%B9%E5%8D%95%E8%A1%A8%E5%90%84%E7%A7%8D%E6%9F%A5%E8%AF%A2%E6%93%8D%E4%BD%9C%E5%8F%8A%E5%87%BD%E6%95%B0%E7%9A%84%E4%BD%BF%E7%94%A83.png)

![](http://7xrbxa.com1.z0.glb.clouddn.com/Database-Oracle-oracleSQL%E5%AF%B9%E5%8D%95%E8%A1%A8%E5%90%84%E7%A7%8D%E6%9F%A5%E8%AF%A2%E6%93%8D%E4%BD%9C%E5%8F%8A%E5%87%BD%E6%95%B0%E7%9A%84%E4%BD%BF%E7%94%A84.png)


> 注意：
> select '123' + 123 from dual;246
> select '123' || 123 from dual;123123



 

