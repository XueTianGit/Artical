title: Oracle-PlSQL程序设计
date: 2016/4/11 8:45:55  
categories: Database
---

# Oracle-PlSQL程序设计 #

## 概述 ##


> 1）PlSQL是专用于Oracle服务器，在SQL基础之上，添加了一些过程化控制语句，叫PLSQL
   过程化包括有：类型定义，判断，循环，游标，异常或例外处理。。。**PLSQL强调过程**。

> 2）使用PLSQL的原因：
因为SQL是第四代命令式语言，无法显示处理过程化的业务，所以得用一个过程化程序设计语言来弥补SQL的不足之处，
SQL和PLSQL不是替代关系，是弥补关系	

> 3）PLSQL与SQL执行有什么不同：
（1）SQL是单条执行的
（2）PLSQL是整体执行的，不能单条执行，整个PLSQL结束用/，其中每条语句结束用；号

PLSQL程序完整组成结构：

	 [declare]
	      变量声明;。。。
	 begin
	      DML/TCL操作;
	  	  DML/TCL操作;
	 [exception]
	      例外处理;
	  例外处理;
	 end;
	 /		

注意：在PLSQL程序中，；号表示每条语句的结束，/表示整个PLSQL程序结束

简单范例：

	declare
	    --定义变量
	    mysum number(3) := 0;
	    tip varchar2(10) := '结果是';
	begin
	    /*业务算法*/   
	    mysum := 10 + 100;
	    /*输出到控制器*/
	    dbms_output.put_line(tip || mysum);
	end;
	/
> dbms_output是oracle中的一个输出对象
> put_line是上述对象的一个方法，用于输出一个字符串自动换行 
> 默认情况下（SQLPlus、SQLDevelper），不显示PLSQL程序的执行结果，我们需要设置，语法：set serveroutput on/off;

## 变量 ##

### 定义变量 ###

	declare
	    --定义变量
		i number(2);
	    mysum number(3) := 0;    
	    tip varchar2(10) := '结果是';
	    pename emp.ename%type;   --pename变量与emp表的ename字段的类型相同
	    emp_record emp%rowtype;  --emp_record与emp表的结构相同，即封装了emp表的每个字段
	begin。。。。。

> 何时使用%type，何时使用%rowtype？

>- 当定义变量时，该变量的类型与表中某字段的类型相同时，可以使用%type
>- 当定义变量时，该变量与整个表结构完全相同时，可以使用%rowtype，此时通过变量名.字段名，可以取值变量中对应的值  例：emp_record.sal项目中，常用%type

### 程序中给变量赋值 ###
.... into 变量..

利用上面定义的pename和emp_record变量做演示：

	....  --省略
	begin  
		select ename into pename from where empo=7369;  -- 将ename的值放入pename变量中
	    select * into emp_record from emp where empno = 7788;	--将*代表的所有字段的内容放入emp_record， 当然必须一一对应的
	end;
	/

## 程序控制语句 ##

### if ###

	格式1：
	IF   条件  THEN 语句1;
	语句2;
	END    IF;
	
	格式2：
	IF  条件  THEN  语句序列1；   
	ELSE   语句序列 2；
	END    IF；


范例：

	declare
	    pday varchar2(10);
	begin
	    select to_char(sysdate,'day') into pday from dual;
	    dbms_output.put_line('今天是'||pday);
	    if pday in ('星期六','星期日') then
		dbms_output.put_line('休息日');
	    else
		dbms_output.put_line('工作日');
	    end if;
	end;
	/


格式3：

	IF   条件  THEN 语句;
	ELSIF  语句  THEN  语句;
	ELSE    语句;
	END   IF;

范例：
从键盘接收值age，依据不同的age进行处理

	declare
	    age number(3) := &age;
	begin
	    if age < 16 then
	       dbms_output.put_line('豆蔻年华');
	    elsif age < 30 then
	       dbms_output.put_line('而立了吗');
	    elsif age < 60 then
	       dbms_output.put_line('快古稀了啊');
	    elsif age < 80 then 
	       dbms_output.put_line('保存健康，长鸣百岁');
	    else
	       dbms_output.put_line('你还在吗？');
	    end if;
	end;
	/

### 循环 ###
PLSQL中共有3种循环格式：

	格式1：
	
	WHILE  condition  
	LOOP
	
	END  LOOP;
	
	格式2：
	Loop
	   exit [when 条件成立];
	   -- .... 
	end loop;

范例：
使用loop循环显示1-10

	declare
	    i number(2) := 1;
	begin
	    loop
	        --当i>10时，退出循环
	        exit when i>10;
	        --输出i的值
	        dbms_output.put_line(i);
	        --变量自加
	        i := i + 1;  
	    end loop;
	end;
	/

	格式3：
	FOR 变量 IN 起始值 . . 结束值  
	LOOP   
	语句序列 ;
	END    LOOP ; 

    //对于for循环来说， 步长是固定的， 每次加1，自动判断条件退出

范例：

	使用for循环显示20-30
	declare
	    i number(2) := 20;
	begin
	    for i in 20 .. 30
	    loop
	        dbms_output.put_line(i);
	    end loop;
	end;
	/

## PLSQL游标Cursor ##
> PLSQL的游标类似于JDBC中的ResultSet，从上向下依次获取每一记录的内容

	1）定义游标：
		CURSOR  游标标名  [ (参数名  数据类型[,参数名 数据类型]...)]
		IS  SELECT   语句；
		例如：cursor c1 is select ename from emp;
	
	2）游标的使用
	1）游标使用前必须打开，使用完要关闭！！，使用fetch关键字来移动游标（类似ResultSet.next()）
	2）游标每次中都按类型保存着其所持有的查询值（查询无果就没值呗）
	3）cemp%notfound  可以判断游标所指行记录是否有值

使用范例（无参游标）：

	declare
	    --定义游标
	    cursor cemp is select ename,sal from emp;
	    --定义变量
	    vename emp.ename%type;
	    vsal   emp.sal%type;
	begin
	    --打开游标，这时游标位于第一条记录之前
	    open cemp;
	    --循环
	    loop
	       --向下移动游标一次
	       fetch cemp into vename,vsal;   --并将游标中的值赋值给vename、vsal
	       --退出循环,当游标下移一次后，找不到记录时，则退出循环
	       exit when cemp%notfound;      
	       --输出结果
	       dbms_output.put_line(vename||'--------'||vsal);
	    end loop;
	    --关闭游标
	    close cemp;
	end;
	/

### 带参游标 ###
> 上面的游标并没有带参数， 使用带参游标我们可以，我们可以给查询语句条件动态赋值。

> 范例：使用带参光标cursor，查询10号部门的员工姓名和工资

	declare
	    cursor cemp(pdeptno emp.deptno%type) is select ename,sal from emp where deptno=pdeptno;
	    pename emp.ename%type;
	    psal emp.sal%type; 
	begin 
	    open cemp(&deptno);
	    loop
	        fetch cemp into pename,psal;	 
	        exit when cemp%notfound;
	        dbms_output.put_line(pename||'的薪水是'||psal);
	    end loop;
	    close cemp;
	end;
	/


## PLSQL例外 ##
类似于java中的异常，用来增强程序的健壮性和容错性。

	Oracle数据库服务器已经定义了一些系统例外:
	no_data_found          (没有找到数据)
	too_many_rows          (select …into语句匹配多个行) 
	zero_Divide            ( 被零除)
	value_error            (算术或转换错误)
	timeout_on_resource    (在等待资源时发生超时)

系统例外范例：

	declare
	    myresult number;
	begin
	    myresult := 1/0;
	    dbms_output.put_line(myresult);
	exception
	    when zero_divide then 
		 dbms_output.put_line('除数不能为0');
		 delete from emp;  
	end;
	/

那么如何自定义例外呢？

	定义语法：
		例外名 exception
	
	抛出例外 ::
	1） raise 例外名;   （抛出用户自定义例外）
	2） raise_application_error('-20000', '用户自定义例外信息');
		该方法可以抛出一个意外  -20000到-20200代表用户自定义例外信息
	
	在exception中接收例外：
	exception
	    when 例外名 then 
		.....
	end;
	/
	
	
	范例：
	declare 
		out_of exception；
		myNumber number(2) := &myNumber；
	begin
		if myNumber=99 then
		raise out_of;
		end if; 
	exception
		when out_of then
		dbms_output.put_line('太多了');
	end;
	/

















