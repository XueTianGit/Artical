title: Spring-AOP编程
date: 2016/3/9 15:47:24   
categories: WebFrame
---

# Spring-AOP编程 #

## AOP编程 ##

> AOP aspect object programming  面向切面编程。它的目标是->实现关注点代码与业务代码分离！
> AOP编程的实现依赖代理。（这样才能动态的添加代码） 

### 专业术语 ###
> - 关注点:重复代码就叫做关注点。
> - 切面:
> 关注点形成的类，就叫切面(类)。  （->重复代码聚集成切面）
> 面向切面编程，就是指 对很多功能都有的重复的代码抽取，再在运行的时候网业务方法上动态植入“切面类代码”。、
> 
> - 切入点（切入关注点代码的地方）
> 执行目标对象方法，动态植入切面代码。
> 可以通过切入点表达式，指定拦截哪些类的哪些方法； 给指定的类在运行的时候植入切面类代码。

## SpringAOP编程的实现方式 ##

### 注解方式实现AOP编程 ###

> 先引入aop相关jar文件    		
				
	spring-aop-3.2.5.RELEASE.jar   【位于spring3.2源码】
	aopalliance.jar				   【spring2.5源码/lib/aopalliance】
	aspectjweaver.jar			   【spring2.5源码/lib/aspectj】或【aspectj-1.8.2\lib】
	aspectjrt.jar				   【spring2.5源码/lib/aspectj】或【aspectj-1.8.2\lib】
	Spring的AOP编程实现依赖的是aspectj组件。


> NT： 用到spring2.5版本的jar文件，如果用jdk1.7可能会有问题。这时应需要升级aspectj组件，即使用aspectj-1.8.2版本中提供jar文件提供。

> bean.xml中引入aop名称空间

	<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">


> 开启aop注解

	<!-- 开启注解扫描  即指定扫描的包-->
	<context:component-scan base-package="com.suixin.e_aop_anno"></context:component-scan>

> 使用注解

	@Aspect	   指定一个类为切面类		
	
	@Pointcut("execution(* com.suixint.e_aop_anno.*.*(..))")    
	指定切入点表达式
	
	@Before("pointCut_()")				    前置通知: 目标方法之前执行
	@After("pointCut_()")					后置通知：目标方法之后执行（始终执行）
	@AfterReturning("pointCut_()")		    返回后通知： 执行方法结束前执行(异常不执行)
	@AfterThrowing("pointCut_()")			异常通知:  出现异常时候执行
	@Around("pointCut_()")				    环绕通知： 环绕目标方法执行

代码范例：

    // 1. bean.xml (开启注解扫描)  
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xmlns:p="http://www.springframework.org/schema/p"
	    xmlns:context="http://www.springframework.org/schema/context"
	    xmlns:aop="http://www.springframework.org/schema/aop"
	    xsi:schemaLocation="
	        http://www.springframework.org/schema/beans
	        http://www.springframework.org/schema/beans/spring-beans.xsd
	        http://www.springframework.org/schema/context
	        http://www.springframework.org/schema/context/spring-context.xsd
	        http://www.springframework.org/schema/aop
	        http://www.springframework.org/schema/aop/spring-aop.xsd">
		
		<!-- 开启注解扫描 -->
		<context:component-scan base-package="com.suixin.e_aop_anno"></context:component-scan>
		
		<!-- 开启aop注解方式 -->
		<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
	</beans>   


   
	// 2 目标对象接口（有接口的话，Spring会使用动态代理实现AOP）
	public interface IUserDao {
		void save();
	}
		  
	/**
	 * 3 目标对象（我们要为这个对象生成代理，实现AOP编程）
	 *
	 */
	@Component   // 加入容器
	public class UserDao implements IUserDao{
	
		@Override
		public void save() {
			System.out.println("-----核心业务：保存！！！------"); 
		}
	}



    //4 切面类（重复的代码）
	@Component
	@Aspect  // 指定当前类为切面类
	public class Aop {

	// 指定切入点表单式： 拦截哪些方法； 即为哪些类生成代理对象
	@Pointcut("execution(* com.suixin.e_aop_anno.*.*(..))")
	public void pointCut_(){
	}
	
	// 前置通知 : 在执行目标方法之前执行
	@Before("pointCut_()")
	public void begin(){
		System.out.println("开始事务/异常");
	}
	
	// 后置/最终通知：在执行目标方法之后执行  【无论是否出现异常最终都会执行】
	@After("pointCut_()")
	public void after(){
		System.out.println("提交事务/关闭");
	}
	
	// 返回后通知： 在调用目标方法结束后执行 【出现异常不执行】
	@AfterReturning("pointCut_()")
	public void afterReturning() {
		System.out.println("afterReturning()");
	}
	
	// 异常通知： 当目标方法执行异常时候执行此关注点代码
	@AfterThrowing("pointCut_()")
	public void afterThrowing(){
		System.out.println("afterThrowing()");
	}
	
	// 环绕通知：环绕目标方式执行
	@Around("pointCut_()")
	public void around(ProceedingJoinPoint pjp) throws Throwable{
		System.out.println("环绕前....");
		pjp.proceed();  // 执行目标方法
		System.out.println("环绕后....");
	}
	
	  
	// 4测试 App.java	   
	
	public class App {
		
		ApplicationContext ac = 
			new ClassPathXmlApplicationContext("com/suixin/e_aop_anno/bean.xml");
	
		// 目标对象有实现接口，spring会自动选择“JDK代理”
		@Test
		public void testApp() {
			IUserDao userDao = (IUserDao) ac.getBean("userDao");
			System.out.println(userDao.getClass());
			userDao.save();
		}
		
		// 目标对象没有实现接口， spring会用“cglib代理”
		@Test
		public void testCglib() {
			OrderDao orderDao = (OrderDao) ac.getBean("orderDao");
			System.out.println(orderDao.getClass());
			orderDao.save();
		}
	}
		 


###  XML方式实现AOP编程 ###

Xml实现aop编程步骤：

- 1） 引入jar文件  【aop 相关jar， 4个】
- 2） 引入aop名称空间
- 3） aop 配置



> 配置切面类 （重复执行代码形成的类）

> aop配置   （拦截哪些方法 / 拦截到方法后应用通知代码）

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xmlns:p="http://www.springframework.org/schema/p"
	    xmlns:context="http://www.springframework.org/schema/context"
	    xmlns:aop="http://www.springframework.org/schema/aop"
	    xsi:schemaLocation="
	        http://www.springframework.org/schema/beans
	        http://www.springframework.org/schema/beans/spring-beans.xsd
	        http://www.springframework.org/schema/context
	        http://www.springframework.org/schema/context/spring-context.xsd
	        http://www.springframework.org/schema/aop
	        http://www.springframework.org/schema/aop/spring-aop.xsd">
		
		<!-- dao 实例 -->
		<bean id="userDao" class="com.suixin.f_aop_xml.UserDao"></bean>
		<bean id="orderDao" class="com.suixin.f_aop_xml.OrderDao"></bean>
		
		<!-- 切面类 -->
		<bean id="aop" class="com.suixin.f_aop_xml.Aop"></bean>
		
		<!-- Aop配置 -->
		<aop:config>
			<!-- 定义一个切入点表达式： 拦截哪些方法 -->
			<aop:pointcut expression="execution(* com.suixin.f_aop_xml.*.*(..))" id="pt"/>
			<!-- 切面 -->
			<aop:aspect ref="aop">
				<!-- 环绕通知 -->
				<aop:around method="around" pointcut-ref="pt"/>
				<!-- 前置通知： 在目标方法调用前执行 -->
				<aop:before method="begin" pointcut-ref="pt"/>
				<!-- 后置通知： -->
				<aop:after method="after" pointcut-ref="pt"/>
				<!-- 返回后通知 -->
				<aop:after-returning method="afterReturning" pointcut-ref="pt"/>
				<!-- 异常通知 -->
				<aop:after-throwing method="afterThrowing" pointcut-ref="pt"/>			
			</aop:aspect>
		</aop:config>
	</beans>  	 

### 切入点表达式深入了解 ###
> 深入了解，深入了解。。。
> 其实没什么深入不深入的， 就是仔细看一下到底怎么写这个切入点表达式（随心所欲的拦截方法）。

spring文档给出的pointcut表达式的定义：

	execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern)  throws-pattern?)


切入点表达式：  可以对指定的“方法”进行拦截；  从而给指定的方法所在的类生层代理对象。
 	
	<aop:config>
		
		<!-- 定义一个切入点表达式： 拦截哪些方法 -->
		<!--<aop:pointcut expression="execution(* com.suixin.g_pointcut.*.*(..))" id="pt"/>-->
		
		<!-- 【拦截所有public方法】 -->
		<!--<aop:pointcut expression="execution(public * *(..))" id="pt"/>-->
		
		<!-- 【拦截所有save开头的方法 】 -->
		<!--<aop:pointcut expression="execution(* save*(..))" id="pt"/>-->
		
		<!-- 【拦截指定类的指定方法, 拦截时候一定要定位到方法】 -->
		<!--<aop:pointcut expression="execution(public * com.suixin.g_pointcut.OrderDao.save(..))" id="pt"/>-->
		
		<!-- 【拦截指定类的所有方法】 -->
		<!--<aop:pointcut expression="execution(* com.suixin.g_pointcut.UserDao.*(..))" id="pt"/>-->
		
		<!-- 【拦截指定包，以及其自包下所有类的所有方法】 -->
		<!--<aop:pointcut expression="execution(* cn..*.*(..))" id="pt"/>-->
		
		<!-- 【多个表达式】 -->
		<!--<aop:pointcut expression="execution(* com.suixin.g_pointcut.UserDao.save()) || execution(* com.suixin.g_pointcut.OrderDao.save())" id="pt"/>-->
		<!--<aop:pointcut expression="execution(* com.suixin.g_pointcut.UserDao.save()) or execution(* com.suixin.g_pointcut.OrderDao.save())" id="pt"/>-->

		<!-- 下面2个且关系的，没有意义 -->
		<!--<aop:pointcut expression="execution(* com.suixin.g_pointcut.UserDao.save()) &amp;&amp; execution(* com.suixin.g_pointcut.OrderDao.save())" id="pt"/>-->
		<!--<aop:pointcut expression="execution(* com.suixin.g_pointcut.UserDao.save()) and execution(* com.suixin.g_pointcut.OrderDao.save())" id="pt"/>-->
		
		<!-- 【取非值】 -->
		<!--<aop:pointcut expression="!execution(* com.suixin.g_pointcut.OrderDao.save())" id="pt"/>-->
		<aop:pointcut expression=" not execution(* com.suixin.g_pointcut.OrderDao.save())" id="pt"/>
		
		<!-- 切面 -->
		<aop:aspect ref="aop">
			<!-- 环绕通知 -->
			<aop:around method="around" pointcut-ref="pt"/>
		</aop:aspect>
	</aop:config>

#### 另一种屌屌的写法 ####

	< !-- 扫描以Service结尾的bean -->
	<aop:pointcut id="serviceOperation" expression="bean(*Service)"/>

   	 
   	 
