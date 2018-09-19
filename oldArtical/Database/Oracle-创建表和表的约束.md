title: Oracle-创建表和表的约束
date: 2016/4/13 8:45:55  
categories: Database
---

# Oracle-创建表和表的约束 #

## 创建表 ##
> 依据SQL99标准， oracle创建表的语法当然和MySQL等数据库差不多。
> 那么我们在创建表时注意的应该就是Oracle的类型了。

	先简单创建一个表：
	drop table if exists users;
	create table users(
	   id int(5) auto_increment primary key,
	   name varchar(4) not null,
	   birthday date default '2015-4-27'
	);

	对表进行重命名：
	rename emp to emps;

->  依据其他表结构，创建emp表的结构，但不会插入数据
 
	create table emp
	as
	select * from xxx_emp where 1<>1;

->  创建表时复制其他表的内容

	create table emp
	as
	select * from xxx_emp where 1=1;   //	注意：where不写的话，默认为true

	依据xxx_emp表，只创建emp表，但不复制数据，且emp表只包括empno,ename字段
	create table emp(empno,ename)
	as
	select empno,ename from xxx_emp where 1=2;
->  摧毁表的内容，但不改变表的结构

	truncate table emp;


### Orale的数据类型 ###
> Oracle的数据类型很多：VARCHAR2、NVARCHAR2、NUMBER、DATE、TIMESTAMP 、LONG。。
> 具体去查看Oracle的文档。
> 这里拿出来几个常用的探讨一下。

	1）NUMBER(p,s)
	文档中是这样描述的：Number having precision p and scale s. The precision p can range from 1 to 38. The scale s can range from -84 to 127.
	
	拿我们比较常用的来说：
	example1：   number(5)  ->  5表示最多存99999   
	example2：   number(6,2)  这个就有点意思了：
	其中2表示最多显示2位小数，采用四舍五入，不足位数补0，   （在sqlplus中同时要设置col ... for ...）， 其实就是和显示有关 
	
	其中6表示小数+整数不多于6位 -> 其中整数位数不得多于4位，可以等于4位
	
	2)VARCHAR2(size)
	拿varchar2(8)来说->  8表示字节, 即不同的编码，存的数据时不一样的， 例如使用GBK编码， 那么最多就存4个字符了。
	> 这和MySQL是不同的， MySQL的varchar(8)中的8表示的是字符。
	
	3）DATE
	即日期， 官方文档给的范围是：January 1, 4712 BC to December 31, 9999 AD （不要问我什么意思， 我也不知道，其实也没必要知道！）
	但是，日期的默认格式是这种：'27-4月-15'
	
	4）CLOB【Character Large OBject】：大文本对象，即超过65565字节的数据对象，最多存储4G
	5）BLOB【Binary Large OBject】：大二进制对象，即图片，音频，视频，最多存储4G

### 操作表的命令 ###
在sqlplus中：

	将表放入回收站
	drop table users;
	
	查询回收站中的对象
	show recyclebin;
	
	闪回，即将回收站还原
	flashback table 表名 to before drop;
	flashback table 表名 to before drop rename to  新表名;
	
	彻底删除users表
	drop table users purge;
	
	清空回收站
	purge recyclebin;


#### drop table 和 truncate table 和 delete from 区别： ####
	drop table
	1)属于DDL
	2)不可回滚
	3)不可带where
	4)表内容和结构删除
	5)删除速度快
	
	truncate table
	1)属于DDL
	2)不可回滚
	3)不可带where
	4)表内容删除
	5)删除速度快
	
	delete from
	1)属于DML
	2)可回滚
	3)可带where
	4)表结构在，表内容要看where执行的情况
	5)删除速度慢,需要逐行删除


## 修改表的结构  alter##
依据SQL99标准，和MySQL等数据库都差不多。
	
	增加列：
	为emp表增加image列，alter table 表名 add 列名 类型(宽度) 
	alter table emp add image blob；
	
	修改列：
	修改ename列的长度为20个字节，alter table 表名 modify 列名 类型(宽度) 
	alter table emp modify ename varchar2(20);
	
	删除列：
	alter table emp drop column image;
	
	重命名列：
	alter table emp rename column ename to username;

### 修改表需要注意的地方 ###
	1）修改表时，不会影响表中原有的数据（我们在没设置数据的情况下）
	2）修改表不可回滚	
	
	经典的笔试题：有【1000亿】条会员记录，如何用最高效的方式将薪水字段清零，其它字段内容不变？
	第一：从emp表中删除sal字段
	      alter table emp 
	      drop column sal;      
	第二：向emp表中添加sal字段，且内容默认0
	      alter table emp
	      add sal number(6) default 0;

## 表的约束 ##
依据SQL99标准，和MySQL等数据库都差不多。
	
	例如：primary key、not null、unique、default、foreign key
	范例：
	create table orders(
	  id number(3) primary key,
	  isbn varchar2(6) not null unique,
	  price number(3) not null,
	  cid number(3),
	  --constraint cid_FK foreign key(cid) references customers(id) on delete cascade 
	  constraint cid_FK foreign key(cid) references customers(id) on delete set null  
	);

由于这个表的字段可能会被其他表引用， 这里在外键列上用到了级联删除,on delete cascade  / on delete set null  


### Oracle特有约束：check ###
简单的说，就是在插入数据时，检查一下这个数据是不是满足我们定的约束条件。。。。感觉是废话。。。。
拿个例子来说吧：

	创建表students，包括id,name,gender,salary字段，使用check约束【性别只能是男或女，薪水介于6000到8000之间】
	create table students(
	  id number(3) primary key,
	  name varchar2(4) not null unique,
	  gender varchar2(2) not null check ( gender in ('男','女') ),
	  salary number(6) not null check ( salary between 6000 and 8000 )
	);

这下懂了不。。。。
这样插就是错的：  insert into students(id,name,gender,salary) values(1,'哈哈','中',6000);   **错**








