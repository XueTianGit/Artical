title: Spring-IOC容器的配置
date: 2016/3/9 15:47:34   
categories: WebFrame
---

# Spring-IOC容器的配置 #

SpringIOC容器，是spring核心内容。它的主要作用是创建对象、处理对象的依赖关系。

## 创建对象 ##

方式：

> - 调用无参数构造器
> - 带参数构造器
> - 工厂创建对象
> 	- 工厂类，静态方法创建对象
> 	- 工厂类，非静态方法创建对象

详细配置：
 
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
	
		<!-- ###############对象创建############### -->
		
		<!-- 1. 默认无参数构造器 -->
		<bean id="user1" class="com.suixin.b_create_obj.User"></bean>
	
		
		<!-- 2. 带参数构造器 -->
		<bean id="user2" class="com.suixin.b_create_obj.User">
			<constructor-arg index="0" type="int" value="100"></constructor-arg>
			<constructor-arg index="1" type="java.lang.String" value="Jack"></constructor-arg>
		</bean>
		
		<bean id="user3" class="com.suixin.b_create_obj.User">
			<constructor-arg index="0" type="int" value="100"></constructor-arg>
			<constructor-arg index="1" type="java.lang.String" ref="str"></constructor-arg>
		</bean>
		
		
		<!-- 3. 工厂类创建对象 -->
		<!-- # 3.1 工厂类，实例方法 -->
		<!-- 先创建工厂 -->
		<bean id="factory" class="com.suixin.b_create_obj.ObjectFactory"></bean>
		<!-- 在创建user对象，用factory方的实例方法 -->
		<bean id="user4" factory-bean="factory" factory-method="getInstance"></bean>
		
		
		<!-- # 3.2 工厂类： 静态方法 -->
		<!-- 
			class 指定的就是工厂类型
			factory-method  一定是工厂里面的“静态方法”
		 -->
		<bean id="user" class="com.suixin.b_create_obj.ObjectFactory" factory-method="getStaticInstance"></bean>
	
	</beans>      

## 处理对象的依赖关系 ##
即Spring中，如何给对象的属性赋值?  【DI, 依赖注入】

- 1) 通过构造函数   ->  就像上面一样
- 2) 通过set方法给属性注入值
- 3）内部bean
- 4) p名称空间
- 5) 自动装配(了解)
- 6) 注解
  	 
###  (常用)Set方法注入值
 
即对象的属性必须提供setXXX方法。

配置文件中的配置：

	<!-- dao instance -->
	<bean id="userDao" class="com.suixin.c_property.UserDao"></bean>

	<!-- service instance -->
	<bean id="userService" class="com.suixin.c_property.UserService">
		<property name="userDao" ref="userDao"></property>  <!-- 注入了userDao对象-->
	</bean>
	
	<!-- action instance -->
	<bean id="userAction" class="com.suixin.c_property.UserAction">
		<property name="userService" ref="userService"></property>
	</bean>	 


### 内部bean
 
	<bean id="userAction" class="com.suixin.c_property.UserAction">
		<property name="userService">
			<bean class="com.suixin.c_property.UserService">
				<property name="userDao">
					<bean class="com.suixin.c_property.UserDao"></bean>
				</property>
			</bean>
		</property>
	</bean>	 


# p 名称空间注入属性值 (优化)

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
	
		<!-- ###############对象属性赋值############### -->
		
		<!-- 
			给对象属性注入值：
				# p 名称空间给对象的属性注入值
				 (spring3.0以上版本才支持)
		 -->
		 <bean id="userDao" class="com.suixin.c_property.UserDao"></bean>
		 
		 <bean id="userService" class="com.suixin.c_property.UserService" p:userDao-ref="userDao"></bean>
		 
		 <bean id="userAction" class="com.suixin.c_property.UserAction" p:userService-ref="userService"></bean>
		
		
		<!-- 传统的注入： 
		 <bean id="user" class="com.suixin.c_property.User" >
		 	<property name="name" value="xxx"></property>
		 </bean>
		-->
		<!-- p名称空间优化后 -->
		<bean id="user" class="com.suixin.c_property.User" p:name="Jack0001"></bean>
	 
	</beans>   	 


### 自动装配(了解)
####=根据名称自动装配：autowire="byName"
	- 自动去IOC容器中找与属性名同名的引用的对象，并自动注入
 
	<!-- ###############自动装配############### -->  
	<bean id="userDao" class="com.suixin.d_auto.UserDao"></bean>	
	<bean id="userService" class="com.suixin.d_auto.UserService" autowire="byName"></bean>

	<!-- 根据“名称”自动装配： userAction注入的属性，会去ioc容器中自动查找与属性同名的对象 -->
	<bean id="userAction" class="com.suixin.d_auto.UserAction" autowire="byName"></bean>	 


也可以定义到全局， 这样就不用每个bean节点都去写autowire=”byName”
 
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd" default-autowire="byName">   根据名称自动装配（全局）
		
		<!-- ###############自动装配############### -->  
		<bean id="userDao" class="com.suixin.d_auto.UserDao"></bean>	
		<bean id="userService" class="com.suixin.d_auto.UserService"></bean>
		<bean id="userAction" class="com.suixin.d_auto.UserAction"></bean>
	</beans>   	 


####根据类型自动装配：autowire="byType"

> 即根据属性的类型，自动去IOC容器中寻找，并且注入;
> NT:
> 必须确保改类型在IOC容器中只有一个对象；否则报错。
> 还是定义成全局：

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd" default-autowire="byType">
	
		<!-- ###############自动装配############### -->  
		<bean id="userDao" class="com.suixin.d_auto.UserDao"></bean>	
	
		<bean id="userService" class="com.suixin.d_auto.UserService"></bean>  
		
		<!-- 如果根据类型自动装配： 必须确保IOC容器中只有一个该类型的对象 -->
		<bean id="userAction" class="com.suixin.d_auto.UserAction"></bean>   //这里会注入UserService对象，但不知道注入哪一个!!!!
		
		
		<!--   报错： 因为上面已经有一个该类型的对象，且使用了根据类型自动装配
		<bean id="userService_test" class="com.suixin.d_auto.UserService" autowire="byType"></bean>
		 -->
	</beans>  	 

总结：
	Spring提供的自动装配主要是为了简化配置； 但是不利于后期的维护。(一般不推荐使用)


### 注解
注解方式可以简化spring的IOC容器的配置!

	使用注解步骤：
	1）先引入context名称空间
		xmlns:context="http://www.springframework.org/schema/context"
	2）开启注解扫描, 即扫描的范围内注解
		<context:component-scan base-package="com.suixin.e_anno2"></context:component-scan>
	3）使用注解
		通过注解的方式，把对象加入ioc容器。
		创建对象以及处理对象依赖关系，相关的注解：
		@Component    指定把一个对象加入IOC容器
		@Repository   作用同@Component； 在持久层使用
		@Service      作用同@Component； 在业务逻辑层使用
		@Controller   作用同@Component； 在控制层使用 
		@Resource     属性注入

总结：

	1） 使用注解，可以简化配置，且可以把对象加入IOC容器,及处理依赖关系(DI)
	2） 注解可以和XML配置一起使用。

	范例：
	NT：记得开启注解扫描
	
	// 把当前对象加入ioc容器
	@Component("userDao")   //  相当于bean.xml 【<bean id=userDao class=".." />】，这里当然也可以这样写@Repository("userDao")
	//@Component  // 若直接这样写的话： 加入ioc容器的UserDao对象的引用名称， 默认与类名一样， 且第一个字母小写，即和 @Component("userDao") 相同
	public class UserDao {}
	
	@Component("userService")  // userService加入ioc容器
	public class UserService {
		
		// 会从IOC容器中找userDao对象，注入到当前字段
		/*
		 * <bean id="" class=""> 
		 *	  <property name="userDao" ref="userDao" />    @Resource相当于这里的配置
		 * </bean>
		 */
		
		@Resource(name = "userDao")
		private UserDao userDao;
		
		public void save() {
			userDao.save();
		}
	}
	
	//测试
	public class App {
	
		// 创建容器对象
		private ApplicationContext ac = new ClassPathXmlApplicationContext("com/suixin/e_anno/bean.xml");
	
		@Test
		public void testExecuteService() {
			// 从容器中获取Action
			UserService userService = (UserService) ac.getBean("userService");
			userService.save();
		}