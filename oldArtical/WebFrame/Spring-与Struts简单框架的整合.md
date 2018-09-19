title: Spring-与Struts简单框架的整合
date: 2016/3/9 9:19:20  
categories: WebFrame
---

# Spring-与Struts简单框架的整合 #

Spring：负责对象对象创建
Struts：用Action处理请求

整合关键点：
让struts框架action对象的创建，交给spring完成！


## 步骤 ##
1）引入jar文件

	引入Spring、Struts需要的相关jar包。
	NT：需引入spring-web 支持jar包:
			spring-web-3.2.5.RELEASE.jar			【位于Spring源码】
			struts2-spring-plugin-2.3.4.1.jar   【位于Struts源码】

2）配置XML:


    bean.xml ：将action交给Spring创建
	<!-- 指定action多例 -->
	<bean id="userAction" class="cn.itcast.action.UserAction" scope="prototype">
		<property name="userService" ref="userService"></property>
	</bean>



    struts.xml：引用Spring创建的action
	<?xml version="1.0" encoding="UTF-8" ?>
	<!DOCTYPE struts PUBLIC
		"-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
		"http://struts.apache.org/dtds/struts-2.3.dtd">
	
	<struts>
		<package name="user" extends="struts-default">
	
			<action name="user" class="userAction" method="execute">
				<result name="success">/index.jsp</result>
			</action>
	
		</package>
	</struts>

    web.xml		
	<!-- 1. struts配置     核心过滤器： 引入struts功能-->
	<filter>
		<filter-name>struts2</filter-name>
		<filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>struts2</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

	<!-- 2. spring 配置    初始化spring的ioc容器 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/classes/bean-*.xml</param-value>
	</context-param>
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

