title:  Spring-简要了解
date: 2016/3/9 9:19:20  
categories: WebFrame
---

# Spring-简要了解 #

## Spring中用到的一些专业术语的了解 ##

	组件/框架设计：
		侵入式设计
				引入了框架，对现有的类的结构有影响；即需要实现或继承某些特定类。
				例如：	Struts框架
		非侵入式设计
			引入了框架，对现有的类结构没有影响。
			例如：Hibernate框架 / Spring框架
	
	控制反转:
		Inversion on Control , 控制反转 IOC
		对象的创建交给外部容器完成，这个就做控制反转.
		
		依赖注入，  dependency injection 
		处理对象的依赖关系
	
		区别：
	       控制反转， 解决对象创建的问题 【对象创建交给别人】
		   依赖注入，在创建完对象后， 对象的关系的处理就是依赖注入 【通过set方法依赖注入】 
	AOP：
		面向切面编程。切面，简单来说来可以理解为一个类，由很多重复代码形成的类。
		切面举例：事务、日志、权限;

## Spring框架 ##

> 何为Spring框架：
> Spring框架，可以解决对象创建以及对象之间依赖关系的一种框架。且可以和其他框架一起使用；Spring与Struts,  Spring与hibernate
> 即它是一个起到整合（粘合）作用的一个框架。


### Spring的六大功能模块 ###

	1） Spring Core     spring的核心功能： IOC容器, 解决对象创建及依赖关系
	2） Spring Web      Spring对web模块的支持。
						   - 可以与struts整合,让struts的action创建交给spring
					       - spring mvc模式
	3） Spring DAO      Spring 对jdbc操作的支持  【JdbcTemplate模板工具类】
	4） Spring ORM      Spring对orm的支持： 
						 既可以与hibernate整合，【session】
						 也可以使用spring的对hibernate操作的封装
	5）Spring AOP       切面编程（重复代码的抽取）
	6）SpringEE         spring 对javaEE其他模块的支持


### 开发步骤 ###

#### 1-引入Spring的jar包 ####
spring各个版本中：

	在3.0以下的版本，源码有spring中相关的所有包【spring功能 + 依赖包】
		如2.5版本；
	在3.0以上的版本，源码中只有spring的核心功能包【没有依赖包】
		(如果要用依赖包，需要单独下载！)


    源码, jar文件：
	commons-logging-1.1.3.jar             日志
	spring-beans-3.2.5.RELEASE.jar        bean节点
	spring-context-3.2.5.RELEASE.jar      spring上下文节点
	spring-core-3.2.5.RELEASE.jar         spring核心功能
	spring-expression-3.2.5.RELEASE.jar   spring表达式相关表

以上是必须引入的5个jar文件，在项目我们可以把他们配置成用户库

#### 2-配置Spring核心配置文件 ####

> 核心配置文件: applicationContext.xml  

	Spring配置文件：applicationContext.xml 或 bean.xml
	
> 约束参考目录：   spring-framework-3.2.5.RELEASE\docs\spring-framework-reference\htmlsingle\index.html
 
	<beans xmlns="http://www.springframework.org/schema/beans"
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xmlns:p="http://www.springframework.org/schema/p"
	    xmlns:context="http://www.springframework.org/schema/context"
	    xsi:schemaLocation="
	        http://www.springframework.org/schema/beans
	        http://www.springframework.org/schema/beans/spring-beans.xsd
	        http://www.springframework.org/schema/context
	        http://www.springframework.org/schema/context/spring-context.xsd">
	
		 <bean id="user1" class="com.suixin.a_hello.User"></bean>  //配置一个对象，即这个对象有Spring来创建
	</beans>   	 

	bean对象创建的细节
	/**
	 *  对应<bean/>标签的属性
	 * 1) 对象创建： 单例/多例
	 * 	scope="singleton", 默认值， 即 默认是单例	【service/dao/工具类】
	 *  scope="prototype", 多例； 				【Action对象】
	 * 
	 * 2) 什么时候创建?
	 * 	  scope="prototype"  在用到对象的时候，才创建对象。
	 *    scope="singleton"  在启动(容器初始化之前)， 就已经创建了bean，且整个应用只有一个。
	 * 3)是否延迟创建
	 * 	  lazy-init="false"  默认为false,  不延迟创建，即在启动时候就创建对象
	 * 	  lazy-init="true"   延迟初始化， 在用到对象的时候才创建对象
	 *    （只对单例有效）
	 * 4) 创建对象之后，初始化/销毁
	 * 	  init-method="init_user"       【对应对象的init_user方法，在对象创建爱之后执行 】
	 *    destroy-method="destroy_user"  【在调用容器对象的destriy方法时候执行，(容器用实现类)】
	 */

#### 3-从IOC容器中获取Spring创建的对象 ####

> 获取方式：

	public class App {
	
		// 1. 通过工厂类得到IOC容器创建的对象
		@Test
		public void testIOC() throws Exception {
			// 普通创建对象
			// User user = new User();
			
			// 现在，把对象的创建交给spring的IOC容器
			Resource resource = new ClassPathResource("com/suixin/a_hello/applicationContext.xml");  //加载配置文件
			// 创建容器对象(Bean的工厂), IOC容器 = 工厂类 + applicationContext.xml
			BeanFactory factory = new XmlBeanFactory(resource);
			// 得到容器创建的对象
			User user = (User) factory.getBean("user");			
		}
		
		//2. （方便）直接得到IOC容器对象 
		@Test
		public void testAc() throws Exception {
			// 得到IOC容器对象
			ApplicationContext ac = new ClassPathXmlApplicationContext("com/suixin/a_hello/applicationContext.xml");
			// 从容器中获取bean
			User user = (User) ac.getBean("user");
		}
	}	 




