title: MyBatis-初识MyBatis
date: 2016/3/9 9:19:20  
categories: WebFrame
---

# MyBatis-初识MyBatis #

## Mybatis简述 ##
> - 首先回顾一下JDBC与HIbernate：
> - JDBC （JavaDataBaseConnectivity,java数据库连接）是一种用于执行SQL语句的JavaAPI。
> 具体实现都由数据库厂商实现。java持久层的框架都是对JDBC的封装。
> - 优点：简单易学，上手快，非常灵活构建sql，效率高。
> - 缺点：代码繁琐，难以写出高质量的代码（资源的释放，SQL注入安全性等），开发者关注多，又要写业务逻辑，又要关注对象的创建和销毁。

> Hibernate：
> 基于ORM，以HQL语句代替SQL（面向对象的思想），方便理解。
> 但是当使用Hibernate处理的业务复杂，关联多张表时，hql极其难写，效率也低。

**可以说MyBatis就是JDBC与Hibernate的折中，使他们之间的平衡点。**


- MyBatis 本是apache的一个开源项目iBatis, 2010年这个项目迁移到了google code，并且改名为MyBatis 。2013年11月迁移到Github。
- MyBatis是一个支持普通SQL查询，存储过程和高级映射的优秀持久层框架。MyBatis消除了几乎所有的JDBC代码和参数的手工设置以及对结果集的检索封装。
- MyBatis可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO（Plain Old Java Objects，普通的Java对象）映射成数据库中的记录。

## MyBatis的简单使用 ##
> 既然是持久层框架，做的事情还是那些事，只不过代码的实现和策略是不同的。
> 下面来看一下MyBatis是如何使用的。

创建工程，并导入MyBatis相关jar包。

    与Mybatis相关包如下：
	mybatis-3.2.2.jar   核心包
	asm-3.3.1.jar       
	cglib-2.2.2.jar
	commons-logging-1.1.1.jar   log4j相关jar包
	log4j-1.2.17.jar


配置MyBatis核心配置文件

> 名字可自定义， 这里叫mybatis.xml。在核心配置文件中一般都配置：数据库连接信息，并引入其他映射文件等
> 
	<?xml version="1.0" encoding="UTF-8" ?>
	<!DOCTYPE configuration
	PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
	"http://mybatis.org/dtd/mybatis-3-config.dtd">
	<configuration>
	    <environments default="mysql">
	        <environment id="mysql">
	            <transactionManager type="JDBC"/>
	            <dataSource type="POOLED">
	                <property name="driver" value="com.mysql.jdbc.Driver"/>
	                <property name="url" value="jdbc:mysql://localhost:3306/mybatisdb?characterEncoding=UTF-8"/>
	                <property name="username" value="root"/>
	                <property name="password" value="root"/>
	            </dataSource>
	        </environment>
	    </environments>
	    <mappers>
	        <mapper resource="com/suiixn/mybatis/domain/User.xml"/>
	    </mappers>
	</configuration>
	
	*  <environments/>元素下可以配置多个environment。 default属性指明默认使用的环境， 即从这个环境中得到SqlSessionFactory。
	*  即每个 SqlSessionFactory  实例只能选择一个运行环境
	*  对于事务管理器有两种：JDBC|MANAGED。  JDBC则需要我们去手动管理。 MANAGED则将事务的管理交个像Spring这样的容器。
	*  dataSource的类型分为3种：UNPOOLED，POOLED，JNDI
	*  <mappers>标签则用来引入定义的映射SQL语句的文件

->  typeAliases 

	当一个文件中SQL映射非常多时，可以给类型起个别名，来减轻输入。
	例如：
	<typeAliases>
		<typeAlias alias="Author" type="domain.blog.Author"/>
		<typeAlias alias="Blog" type="domain.blog.Blog"/>
		<typeAlias alias="Comment" type="domain.blog.Comment"/>
	</typeAliases>

在这个配置中，您就可以在想要使用"domain.blog.Blog"的地方使用别名“Blog”了。


配置SQL映射XML 
> - SQL 映射XML文件只有一些基本的元素需要配置，并且要按照下面的顺序来定义：
> - cache –在特定的命名空间配置缓存。
> - cache-ref – 引用另外一个命名空间配置的缓存.
> - resultMap – **最复杂也是最强大的元素，用来描述如何从数据库结果集里加载对象。**
> - parameterMap – 不推荐使用! 在旧的版本里使用的映射配置，这个元素在将来可能会被删除
> - sql – 能够被其它语句重用的 SQL 块。
> - insert –INSERT 映射语句
> - update –UPDATE 映射语句
> - delete –DELEETE 映射语句
> - select –SELECT 映射语句

简单范例：

	<mapper namespace="com.suixin.mybatis.domain.User">
		<!-- 查询一条 -->
		<select id="get" parameterType="string" resultType="com.suixin.mybatis.domain.Userr">
			SELECT * FROM user_c WHERE id=#{id}
		</select>
	</mapper>

* 在这个文件中，我们配置了一条SQL语句， 并对SQL语句的查询结果进行了映射。
* resultType我们将查询结果映射到其所指的类型中（User），但是必须要遵循一定的规则， 即这个PO的属性必须和数据库中相应的表的字段一一对应
* namespace属性指明这条被映射的SQL所属空间
* #{}用于动态取值（parameterType得值）

代码书写

		//1. 读取配置文件，获取SessionFactory
		String resource = "sqlMapConfig.xml";	//配置文件
		InputStream in = Resources.getResourceAsStream(resource);
		sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);	//获得sqlSessionFactory，在build时， 可以指定环境。忽略按默认环境构建
		
	    //根据SQL映射文件，进行查询
		SqlSession session = sqlSessionFactory.openSession();		//获得SqlSession
		User u  = session.selectOne(User.class.getName()+".get","3");	//获取一条  ，“3”即是我们传入的参数



- 总结：
从这一遍简单的开发流程来看：
- 1）SQL写在xml里，便于统一管理和优化。
- 2）解除sql与程序代码的耦合。
- 3）可以很简单的对结果进行映射




















