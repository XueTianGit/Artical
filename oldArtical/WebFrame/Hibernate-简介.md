title: Hibernate-简介
date: 2016/3/9 9:19:20  
categories: WebFrame
---

# Hibernate-简介 #

- Hibernate是一个开放源代码的基于**持久层**的**对象关系映射框架**，它对JDBC进行了非常轻量级的对象封装，使得Java程序员可以随心所欲的使用对象编程思维来操纵数据库。
- Hibernate可以应用在任何使用JDBC的场合，既可以在Java的客户端程序使用，也可以在Servlet/JSP的Web应用中使用.
- 最具革命意义的是，Hibernate可以在应用EJB的J2EE架构中取代CMP，完成数据持久化的重任。

## ORM概念 ##

> Object Relation Mapping。即对象关系映射。

ORM想解决什么问题？

	存储：   能否把对象的数据直接保存到数据库？ 
	获取：   能否直接从数据库拿到一个对象？

> Hibernate与ORM的关系： Hibernate是ORM的实现！

## 开发步骤 ##
	1. 下载源码
		版本：hibernate-distribution-3.6.0.Final
	2. 引入jar文件
		hibernate3.jar核心  +  required 目录（必须引入的6个) +  jpa 目录  + 数据库驱动包
	3. 写对象以及对象的映射
		Employee.java            对象
		Employee.hbm.xml        对象的映射 (映射文件)
	4. src/hibernate.cfg.xml  配置主配置文件
		- 数据库连接配置
		- 加载所用的映射(*.hbm.xml)


