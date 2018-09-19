title: SpringMVC-初识SpringMVC
date: 2016/3/9 15:47:43   
categories: WebFrame
---

# SpringMVC-初识SpringMVC #

## 概述 ##
springmvc属于spring框架的后续产品，用在基于MVC的表现层开发，类似于struts2框架。
> SpringMVC是从Spring框架中抽取出来的。SpringMVC等价于Spring web mvc
> 所以SpringMVC是要依赖Spring框架的核心功能。（导包时就可以看出）

对比于Struts2：
> Struts2也是非常优秀的MVC构架，优点非常多比如良好的结构，拦截器的思想，丰富的功能。
> 但这里想说的是缺点，Struts2由于采用了值栈、OGNL表达式、struts2标签库等，会导致应用的性能下降，应避免使用这些功能。


先来看一下Spring的工作流程：

![](http://7xrbxa.com1.z0.glb.clouddn.com/WebFramespringmvc%E5%88%9D%E8%AF%861.png)



从该图大致可以了解：

- 1. SpringMVC中也有类似Struts2的核心拦截器的东西存在， 他是核心Servlet- DispatherServlet（前置控制器）
- 2. http请求经过核心Servlet后会到Controller， Controller则是负责处理请求
- 3. Controller会一model形式封装返回数据，并回到核心Servlet
- 4. 核心servlet会把model传给视图界面去显示数据

**实际上该图还是有一些东西没有画出来的， 就是在核心Servlet和COntroller之间还有**
**映射器、视图器、适配器**

## Spring简单实用 ##
以MyEclipse下的web工程为例：

	1）首先当然是导入jar包（这里使用Spring3.0.5）
	jar包包括Spring核心包、Springweb和springmvc相关的包：
	
	org.springframework.web-3.0.5.RELEASE.jar
	org.springframework.web.servlet-3.0.5.RELEASE.jar（mvc专用）
	   ------------------------------------------------------springIOC模块
	org.springframework.asm-3.0.5.RELEASE.jar
	org.springframework.beans-3.0.5.RELEASE.jar
	org.springframework.context-3.0.5.RELEASE.jar
	org.springframework.core-3.0.5.RELEASE.jar
	org.springframework.expression-3.0.5.RELEASE.jar
	
	
	2）在web.xml中配置Spring核心Servlet
		<servlet>
			<servlet-name>DispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		</servlet>
		<servlet-mapping>
			<servlet-name>DispatcherServlet</servlet-name>
			<url-pattern>*.action</url-pattern>
		</servlet-mapping>

**<url-pattern>  指明给Servlet会处理什么样的请求**

	3）创建Action    (和Struts2保持一致)
		
		public class HelloAction implements Controller{
			/**
			 * 业务方法
			 */
			public ModelAndView handleRequest(HttpServletRequest requqest,HttpServletResponse response) throws Exception {
				ModelAndView modelAndView = new ModelAndView();
				modelAndView.addObject("message","这是我的第一个springmvc应用程序");
				modelAndView.setViewName("/jsp/success.jsp");
				return modelAndView;
			}
		}

	4）创建对应目录下的/jsp/success.jsp页面
	
	5）在/WEB-INF/创建DispatcherServlet-servlet.xml配置文件，xml头部信息与spring.xml相同
	> 为什么叫这个名字呢？  - 该配置文件的命名规则：web.xml文件中配置的<servlet-name>的值-servlet.xml
	
	在这个文件中对Action进行配置：
	    <bean name="/hello.action" class="com.suixin.springmvcdemo.HelloAction"></bean>  

**该Action的访问路径是：http://127.0.0.1:8080/springmvcDemo/hello.action**




## 加载自定义的springmvc配置文件 ##
> 上面的简单实用配置时， DispatcherServlet-servlet.xml  是不是让人感觉很循规蹈矩。 
> 我们当然可以不根据他的规则来， 不过当然得配一下， 不然springmvc怎么知道你想怎么干呢？

我们可以再web.xml中这样配置一个初始化参数：

		<servlet>
			<servlet-name>DispatcherServlet</servlet-name>
			<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
			<init-param>
				<param-name>contextConfigLocation</param-name>
				<param-value>/WEB-INF/classes/com/suixin/springmvcdemo/springmvc.xml</param-value>	
			</init-param>
		</servlet>

这样就可以叫任意名字， 放在任意位置了。

-> 但我们一般都把他放在src目录下， 如果在src下， 可以这样简写：

	<servlet>
		<servlet-name>DispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:spring.xml</param-value>	
		</init-param>
	</servlet>

## 视图解析器 ##

> 说这个之前， 先来看一下ModelAndView对象
### ModelAndView  ###
这个对象，是用来帮助Controller(Action)返回一个模型和视图- 即用来封装模型和视图的。
对应方法有：

	ModelAndView	addObject(String attributeName, Object attributeValue)
	void	setViewName(String viewName)

> 默认时ModelAndView对象中封装的视图路径会安真实路径进行解析。
> 若ModelAndView中封装了逻辑名，那么
> SpringMVC允许我们通过配置视图解析器来将逻辑路径对应的真实路径告诉它（SpringMVC）
> 这里使用InternalResourceViewResolver这个视图解析器

在:springmvc.xml

	   <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	    	<property name="prefix" value="/jsp/"/>
	    	<property name="suffix" value=".jsp"/>
	    </bean>
	
	
	这样我们就可以直接这样进行封装：
				modelAndView.setViewName("success");  


**适配器和映射器后面讨论。**








