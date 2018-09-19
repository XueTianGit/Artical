title: SpringMVC-控制器映射器视图解析器
date: 2016/3/9 15:48:12   
categories: WebFrame
---

# SpringMVC-控制器、映射器、视图解析器 #


首先，在我们没有向Spring容器注册任何控制器、映射器、视图解析器时， 
Spring容器中已经默认提供了控制器、映射器、视图解析器，以便DispatcherServlet使用。
他们是：

	org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping
	org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter  
	org.springframework.web.servlet.view.InternalResourceViewResolver

**这些，我们可以配，也可以不配。如果配了的话就会使用我们的**

## 控制器 Controller##
> 我们将实现了Controller借口的action叫做控制器类。
> Action（Controller）的作用当然是用来处理请求的。
> 当一个请求访问xx.action时， SpringMVC会创建Action实例， 并且这个Action是单例的。

### Action直接实现Controller接口 ###
例如：

	public class EmpAction implements Controller{
		public ModelAndView handleRequest(HttpServletRequest request,HttpServletResponse response) throws Exception {
			ModelAndView  modelAndView = new ModelAndView();
			/*......*/
			return modelAndView;
		}
	}

- 可以看出， 直接实现Controller接口， 我们可以通过request和response获得前台数据， 并且通过ModelAndView来向前台传递数据。
- 不过直接实现Controller接口并不好。因为这样就将传统web代码和SpringMVC混合在一起，耦合太深。


下面来看几个其他Controller
### ParameterizableViewController ###

> 这个控制器可以帮助我们从JSP直接跳到html/jsp，  即中间不需要再经过Action。

例如， 我们想通过点检下面这个超链接，直接跳转到首页

	<a href="${pageContext.request.contextPath}/index.action" style="text-decoration:none">首页</a> 
	->这时就可以配置这个控制器
	<bean name="/index.action" class="org.springframework.web.servlet.mvc.ParameterizableViewController">
    	<!-- 转发到真实视图名 -->
    	<property name="viewName" value="/WEB-INF/05_index.jsp"/>
    </bean>

### AbstractCommandController ###
> 相比于实现Controller接口， 如果继承AbstractCommandController，我们就可以以实体形式来收集请求参数（收集到一个Object对象中）。
使用他的这个功能时，应注意以下几点：

- 如果我们的实体中属性有日期类型的话， 这个控制器是不能自动完成转换的，我们需使用日期转化器
- 对于请求数据封装，它并不能解决前台带过来的数据的乱码问题， 我们需要配置一个编码过滤器（针对POST请求）

	在SpringMVC中，当我们的POST请求携带数据出现乱码时，我们一般都可以通过配置编码过滤器解决。

看如下代码， 前台带过来的字符串形式的日期，我们将他转为Date类型：

	public class EmpAction extends AbstractCommandController{
		
		public EmpAction(){
			//将表单参数封装到Emp对象中去
			this.setCommandClass(Emp.class); 
		}
		/**
		 * 自定义类型转换器，将String->Date类型(格式yyyy-MM-dd)
		 */
		@Override
		protected void initBinder(HttpServletRequest request,ServletRequestDataBinder binder) throws Exception {
			//向springmvc内部注入一个自定义的类型转换器
			//参数一：将String转成什么类型的字节码
			//参数二：自定义转换规则
			//true表示该日期字段可以为空
			binder.registerCustomEditor(
					Date.class,
					new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd"),true));
		}
		/**
		 * obj表示封装后的实体
		 * error表示封装时产生的异常
		 */
		@Override
		protected ModelAndView handle(
				HttpServletRequest request,
				HttpServletResponse response, 
				Object obj, 
				BindException error)throws Exception {
			
			ModelAndView modelAndView = new ModelAndView();
			modelAndView.addObject("message","增加员工成功");
			
			//将Emp封装到ModeAndView对象中
			modelAndView.addObject("emp",emp);	
			modelAndView.setViewName("/jsp/success.jsp");
			return modelAndView;
		}
	}


编码过滤器：

	<filter>
		<filter-name>CharacterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>CharacterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>	


## 映射器Mapping ##
> 首先Action当然是用来处理请求的，请求对于action的具体体现就是请求Action的路径。
那么这个路径是怎么确定的呢？

这就依赖于映射器对Action处理请求进行映射。下面讨论几个SpringMVC常用的映射器：

	1) BeanNameUrlHandlerMapping
	这个映射器，将Action<bean>的name属性映射为Action的访问路径  （这个Action就能处理这一个请求了）
	例如，下面的配置， 访问该action的路径就是/hello.action：
	
	<bean name="/hello.action" class="com.suixin.springmvc-demo.HelloAction"></bean>  
	
	默认情况下，我们可以不配置这个映射器，DispatherServlet（前端控制器）默认会选择他来进行action的寻找。
	你要想配置的话也可以：
	  <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping/>  (可省)
	  
	
	
	2）SimpleUrlHandlerMapping
	这个映射器，允许我们更加自由的配置action去处理多个请求。（多个路径可以访问到Action）
	例如，下面配置我们让 "/delete.action"、"/update.action"、"/find.action"这些请求都被UserAction处理。
	
	 	<bean id="userActionID" class="com.suixin.springmvc-demo.UserAction"></bean>
			
		  <!-- 注册映射器(handler包)(框架) -->
		  <bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
		  		<property name="mappings">
		  			<props>
		  				<prop key="/delete.action">userActionID</prop>
		  				<prop key="/update.action">userActionID</prop>
		  				<prop key="/find.action">userActionID</prop>
		  			</props>
		  		</property>
		  </bean>
	
	3) DefaultAnnotationHandlerMapping
	这个映射器就非常灵活了。我们一般都用这个， 不过要通过注解的方式进行配置。->  @RequestMapping
	
	@RequestMapping(value="/user")   
	public class UserAction {
		@RequestMapping(value="/register.action")
		public String registerMethod(User user,Model model) throws Exception{
				/*.....*/
		}
	
		@RequestMapping(value={"/upadte"，"/upadte2"})
		public String upadteMethod(User user,Model model) throws Exception{
				/*.....*/
		}	
	}
	
	*  value可以接收一个数组。 即多个路径访问这一个方法。
	*  @RequestMapping配置在Action上就类似于Struts2的namespace
	*  registerMethod的访问路径为：  /user/register.action
	*  对于upadteMethod方法的路径配置， 这里并没有写.action（DispatherServlet拦截的类型），这也是可以的


NT：DefaultAnnotationHandlerMapping 在3.1之前都是OK额
	如果使用的SpringMVC是3.1之后的版本， 则可以使用RequestMappingHandlerMapping


## 视图解析器 ##
> 一般就使用他
> org.springframework.web.servlet.view.InternalResourceViewResolver


- 1）它的作用就是解析action返回的视图路径，（Action返回字符串或者ModelAndView都可以）
- 2）如果Action返回真实路径，我们并不需要要配置他
- 3）如果Action返回逻辑路径，那就需要配置了。 


例如,下面代码，Action返回了逻辑路径：

	public class HelloAction implements Controller{
		/**
		 * 业务方法
		 */
		public ModelAndView handleRequest(HttpServletRequest request,HttpServletResponse response) throws Exception {
			ModelAndView modelAndView = new ModelAndView();
			modelAndView.addObject("message","视图使用逻辑名");
			
			//现在封装视图的逻辑名称
			modelAndView.setViewName("success");  //对应真实路径为：/jsp/success.jsp
			return modelAndView;
		}
	}

	 //配置视图解析器
      <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
      		<!-- 路径前缀 -->
      		<property name="prefix" value="/jsp/"/>
      		<!-- 路径后缀 -->
      		<property name="suffix" value=".jsp"/>
      		<!-- 前缀+视图逻辑名+后缀=真实路径 -->
      </bean>





