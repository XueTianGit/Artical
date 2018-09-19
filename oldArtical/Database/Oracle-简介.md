title: Oracle-简介
date: 2016/4/16 8:45:55  
categories: Database
---

# Oracle-简介 #

## oracle概述 ##

-> 一些关于数据库的概念

        数据：在数据库领域看来，数据是存储的基本单位，包含文本，图片，视频，音频
        数据库：就是数据仓库，存储数据的地方，特指计算机设备中的硬盘，以二进制压缩文本的形式存放
               该文件不能直接操作，必须由各数据库公司提供的工具方可操作，该文件的格式是每个数据库公司内部
               定义的，不是统一规则
        数据库对象：在Oracle中，例如：表，视图，索引，函数，过程，触发器。。。
        关系型数据库：简单的说，以行列结构的形式，将数据库中的信息表示出来的对象，即二维表
        常见流行的关系型数据库：Oracle&MySQL/Oracle-->DB2/IBM--->SQLServer/Microsoft-->。。。

-> oracle11g背景	

	1977年 美国人 Larry 成立软件开发实验室 
	1980年 用c/c++开发了世界第一个商用关系型数据库（RDBMS）
	1983年 公司更名为Oracle Corporation（甲骨文公司）
	30多年的发展，Oracle成为世界上领先的信息管理软件供应商和独立软件开发公司，Oracle技术几乎涉及各个行业
	2009年4月21日，原SUN -> Oracle
	市场份额：【Oracle(54%+-)--->IBM-DB2(21%+-)--->MicrosoftMSSQL(14%+-)】	
	
	Oracle认证种类：
	（1）开发技术认证：  Java认证, 数据库开发语言SQL和PL/SQL认证
	（2）数据库技术认证： OCM认证【大师】  OCP认证【专家】 OCA认证【初级】
	（3）中间件技术认证： OracleServer认证，WEB服务器认证，。。。
	（4）专业领域的应用技术认证：   ERP认证  CRM认证 HR认证  OA认证 TAX认证。。
	
	2007年7月12日，甲骨文公司在美国推出Oracle11g，建议至少使用JDK6
	它有400多项功能，经过了1500万个小时的测试，开发工作量达到3.6万人/月
	
	早期：Oracle8i，9i，【10i】----（i表示internet）
	近期：Oracle11g-------------（g表示grid），将多台Oracle服务器当作一台主机协调使用，网格计算被之泛视为未来的计算方式
	现在：oracle 12c(clound融合云计算服务器)
	
	window7/8中，查询端口的命名：netstat -a，或使用其它第三方工具查询当前计算机中的端口号
	oracle数据库的主端口：1521，通常固定不变，前提是：开机时要启动"OracleOraDb11g_home1TNSListener"服务
	
	报价：oracle11g企业版/标准版/个人版-----￥19.5万+-
	报价：oracle12c企业版/标准版/个人版-----￥31.79万+-
	
	
	->oracle数据库服务器由二部份组成
	（A）实例：理解为对象,看不见的
	（B）数据库：理解为类，看得见的，E:\app\Administrator\oradata\orcl\*.DBF
	
	->oracle服务器与orcl数据库的关系
	一个oracle数据库服务器中包括多个数据库，例如：orcl，orm，oa，bbs，tax，erp等等
	例如：在E:\oracleDB\oradata\目录下，有多少个文件夹，就有多少个数据库，例如：orcl文件夹=orcl数据库
	我们向数据库中存储的所有数据库，最终都会存放在对应库的*.DBF文件中，以二进制压缩形式存放 
	注意：我们在安装oracle时，已经创建好了一个数据库，默认名叫orcl，除非你当时改了数据库名字  

### Oracle数据库链接工具 ###
为了链接数据库实例，操作数据库，我们可以使用一些客户端工具。

	1）sqlplus
	sqlplus是oracle11g自带的一个客户端黑屏界面工具，该工具可以连接到某个数据库的实例上，从而操作数据库
	2）sqldeveloper
	sqldeveloper是oracle11g自带的一个客户端彩屏界面工具，该工具可以连接到某个数据库的实例上，从而操作数据库
	3）第三方客户端工具
	如果你觉得这二款客户端工具不喜欢，可以上网下载第三方的客户端工具
	
	->失败转移和负载平衡概念 (不光出现在数据库领域，也能出现在WEB服务器领域)
	失败转移：一个群集中的某个oracle服务器坏掉，应该让该台oracle服务器上的用户转移到其它的几台oracle服务器上,这个过程对用户来说，无需知道
	负载平衡：多个用户来并发访问时，集群内的oracle服务器共同承担用户并发访问的压力，但不一定是平均分配


### 登录Oracle数据库 ###
->使用客户端sqlplus工具进入与退出orcl数据库
>
        ------以超级管管理员角色进入		
        c:/>sqlplus / as sysdba					
        sql>exit    

> 我们登录一次数据库称为一次会话， exit就代表着退出这次会话。

### 用户解锁 ###
> 在我们安装好Oracle数据库后，出sys用户外，其他用户都是锁上的，即不能用，在使用前必须要解锁。例如：以sys超级用户名，dba角色，即超级管理员身份解锁scott方案/用户，并为scott设置一个密码为tiger

以sys登录后

	解锁用户：alter user scott account unlock;
	设置密码：alter user scott identified by tiger; 

这样我们就可以使用scott用户了。
	
	当然在以scott用户登录的情况下我们也是可以更改密码的。
	即使用password命令
	password
	旧口令：tiger
	新口令：abc123
	再次输入新口令：abc123

这样就完成了密码的更改。


## oracleSQL和oracle的关系 ##

	->第四代语言：SQL【结构化查询语言，面向关系的】
		第一代：机器语言
		第二代：汇编		
		第三代：C/C++/C#/Java/VB/...
		第四代：SQL
		 将来。。。
	
	->SQL92/【99】标准的四大分类 
		（A）DML（数据操纵语言）：select,insert,update,delete  
		（B）DDL（数据定义语言）：create table,alter table，drop table，truncate table  。。。
		（C）DCL（数据控制语言）：grant 权限 to scott，revoke 权限 from scott  。。。
		（D）TCL（事务控制语言）：commit，rollback，rollback to savepoint 。。。
	
	
	->oracleSQL与SQL92/99的关系
		SQL92/99标准，访问任何关系型数据库的标准
		oracleSQL语言，只访问Oracle数据库服务器的专用语言
	
	->Java技术和oracleSQL的关系
		JDBC-->使用OracleSQL语法-->Oracle服务器--->orcl数据库-->表-->记录
		Hibernate-->使用OracleSQL语法-->Oracle服务器
		MyBatis-->使用OracleSQL语法-->Oracle服务器

