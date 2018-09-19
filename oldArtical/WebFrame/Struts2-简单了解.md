title: Struts2-简单了解
date: 2016/3/9 15:56:45   
categories: WebFrame
---

# Struts2-简单了解 #

Struts是基于mvc模式的框架， struts其实也是servlet封装，提高了开发效率。
它通过配置的方式，解决了传统MVC模式使用Servlet开发的缺点：
1. 跳转代码写死，不灵活
2. 每次都去写servlet，web.xml中配置servlet！

Struts2 是在Struts1的基础上，融合了xwork的功能;  也可以说，Struts2 = struts1  +  xwork

Struts2框架预先实现了一些功能：
1. 请求数据自动封装
2. 文件上传的功能
3. 对国际化功能的简化
4. 数据效验功能
……………….

## Struts2开发流程 ##


1）引入jar文件（以2.3版本为例）

	commons-fileupload-1.2.2.jar	  
	commons-io-2.0.1.jar           【文件上传相关包】
	struts2-core-2.3.4.1.jar       【struts2核心功能包】
	xwork-core-2.3.4.1.jar         【Xwork核心包】
	ognl-3.0.5.jar				   【Ognl表达式功能支持表】
	commons-lang3-3.1.jar          【struts对java.lang包的扩展】
	freemarker-2.3.19.jar          【struts的标签模板库jar文件】
	javassist-3.11.0.GA.jar        【struts对字节码的处理相关jar】

2）配置web.xml

	NT：Tomcat启动时受限加载自身web.xml，然后加载所有项目的web.xml
	通过在项目的web.xml中引入过滤器，
	->Struts的核心功能的初始化，通过过滤器完成
	 
	<!-- 引入struts核心过滤器 -->
	<filter>
		<filter-name>struts2</filter-name>
		<filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>struts2</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>	   
	 

> struts2-core-2.3.4.1.jar中的StrutsPrepareAndExecuteFilter  即为核心过滤器
> 使用的struts的版本不同，核心过滤器类是不一样的！

3）开发Action

	NT：
	1. action类，也叫做动作类; 一般继承ActionSupport类
	    即处理请求的类  (struts中的action类取代之前的servlet)
	2. action中的业务方法，处理具体的请求
		**- 必须返回String 方法不能有参数**
	 
	public class HelloAction extends ActionSupport {
		
		// 处理请求
		@Override
		public String execute() throws Exception {}  /
	}	 



4)配置struts.xml

>- Struts2执行流程
>- 服务器启动：
> 	- 1. 加载项目web.xml
> 	- 2. 创建Struts核心过滤器对象， 执行filter ->init()方法执行		
>- 当用户访问时：
>   - 3. 用户访问Action, 服务器根据访问路径名称，找对应的aciton配置, 创建action对象
>   - 4. 执行默认拦截器栈中定义的18个拦截器  （Struts2的核心功能是由拦截器实现的）
>   - 5. 执行action的业务处理方法


一些配置文件的含义

	struts-default.xml,    核心功能的初始化
	struts-plugin.xml,     struts相关插件
	struts.xml			   用户编写的配置文件


## struts-default.xml详解 s
目录位置：struts2-core-2.3.4.1.jar/ struts-default.xml

内容：
	1. bean节点指定struts在运行的时候创建的对象类型
	2. 指定struts-default包  -----注意：->用户写的package(struts.xml)一样要继承此包 
	
	package  struts-default  包中定义了：
	a.  跳转的结果类型
		dispatcher    转发，不指定默认为转发
		redirect       重定向
		redirectAction  重定向到action资源
		stream        (文件下载的时候用)
	b. 定义了所有的拦截器
		  定义了32个拦截器！
		  为了拦截器引用方便，可以通过定义栈的方式引用拦截器，
	    此时如果引用了栈，栈中的拦截器都会被引用!
		
		defaultStack
			默认的栈，其中定义默认要执行的18个拦截器！
	
	
	c. 默认执行的拦截器栈、默认执行的action
		<default-interceptor-ref name="defaultStack"/>
	   <default-class-ref class="com.opensymphony.xwork2.ActionSupport" />




>  拦截器什么时候执行？
>  1. 用户访问时候按顺序执行18个拦截器；
> 2. 先执行Action类的创建，再执行拦截器； 最后拦截器执行完，再执行业务方法




