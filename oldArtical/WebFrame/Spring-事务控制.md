title: Spring-声明式事务控制
date: 2016/3/9 9:19:20  
categories: WebFrame
---

# Spring-声明式事务控制 #

对于基于MVC的开发

用户访问 —> Action -> Service -> Dao  (SSH框架)
一个业务的成功： 调用的service是执行成功的，意味着service中调用的所有的dao是执行成功的。  所以： **事务应该在Service层统一控制**。

## 事务控制概述 ##
> 编程式事务控制

	自己手动控制事务，就叫做编程式事务控制。
	Jdbc代码：
		Conn.setAutoCommite(false);  // 设置手动控制事务
	Hibernate代码：
		Session.beginTransaction();    // 开启一个事务
	细粒度的事务控制： 可以对指定的方法、指定的方法的某几行添加事务控制
	(比较灵活，但开发起来比较繁琐： 每次都要开启、提交、回滚.)

> 声明式事务控制

	Spring提供了对事务的管理, 这个就叫声明式事务管理。
	Spring提供了对事务控制的实现。用户如果想用Spring的声明式事务管理，只需要在配置文件中配置即可； 不想使用时直接移除配置。这个实现了对事务控制的最大程度的解耦。
	Spring声明式事务管理，核心实现就是基于Aop。
	粗粒度的事务控制： 只能给整个方法应用事务，不可以对方法的某几行应用事务。->  	(因为aop拦截的是方法。)
	Spring声明式事务管理器类：
	Jdbc技术：DataSourceTransactionManager
	Hibernate技术：HibernateTransactionManager

## Spring声明式事务控制 ##

- 1）首先，得引入spring-aop相关的4个jar文件。
- 2） 引入aop名称空间   【XML配置方式需要引入】
- 3） 引入tx名称空间    【必须引入】

事务控制有两种配置方式： XML配置、注解方式配置

### XML方式的配置 ###

	<!-- #############5. Spring声明式事务管理配置############### -->
		<!-- 5.1 配置事务管理器类 -->
		<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
			<property name="dataSource" ref="dataSource"></property>
		</bean>
		
		<!-- 5.2 配置事务增强(如果管理事务?) -->
		<tx:advice id="txAdvice" transaction-manager="txManager">
			<tx:attributes>
				<tx:method name="get*" read-only="true"/>
				<tx:method name="find*" read-only="true"/>
				<tx:method name="*" read-only="false"/>
			</tx:attributes>
		</tx:advice>
		
		<!-- 5.3 Aop配置： 拦截哪些方法(切入点表表达式) + 应用上面的事务增强配置 -->
		<aop:config>
			<aop:pointcut expression="execution(* com.suixin.a_tx.DeptService.*())" id="pt"/>
			<aop:advisor advice-ref="txAdvice" pointcut-ref="pt"/>
		</aop:config>
		
	</beans>     

这里配置：
对DeptService类的所有方法进行事务管理。即这些方法严格遵循事务操作！！！

### 注解方式的配置 ###

使用注解实现Spring的声明式事务管理，更加简单。

	1） bean.xml中指定注解方式实现声明式事务管理以及应用的事务管理器类
		<tx:annotation-driven transaction-manager="txManager"/>
	
	捎带着别忘了开启注解扫描：
		<context:component-scan base-package="com.suixin.a_tx"></context:component-scan>
	
	2） 在需要添加事务控制的地方，写上: @Transactional


#### @Transactional ####

> @Transactional注解的位置：

	它是应用事务的注解
	1）加到到方法上： 当前方法应用spring的声明式事务
	2）加到到类上：   当前类的所有的方法都应用Spring声明式事务管理;
	3）加到到父类上： 当执行父类的方法时候应用事务。
	
	例如：  下面这个事务就得回滚
			@Transactional
			public void save(Dept dept){
				deptDao.save(dept);
				int i = 1/0;
				deptDao.save(dept);
			}

> @Transactional注解的属性（事务属性）：

例如：

	@Transactional(
			readOnly = false,       // 读写事务
			timeout = -1,           // 事务的超时时间不限制
			noRollbackFor = ArithmeticException.class,   // 遇到数学异常不回滚
			isolation = Isolation.DEFAULT,               // 事务的隔离级别，数据库的默认
			propagation = Propagation.REQUIRED			 // 事务的传播行为
	)



何为事务的传播行为呢？

	事务传播行为:
	Propagation.REQUIRED
		指定当前的方法必须在事务的环境下执行；
		如果当前运行的方法，已经存在事务， 就会加入当前的事务；  （跟在大哥混）
	Propagation.REQUIRED_NEW
		指定当前的方法必须在事务的环境下执行；
		如果当前运行的方法，已经存在事务：  事务会挂起； 会始终开启一个新的事务，执行完后；  刚才挂起的事务才继续运行。（你是你，我是我，我不认识你！！！）


例如：

	Class Log{
			Propagation.REQUIRED  （insertLog()方法加了这个属性）
			insertLog();  
	}
	
		Propagation.REQUIRED
		Void  saveDept(){
			insertLog();    // 加入当前事务
			.. 异常, 会回滚
			saveDept();
		}
	
	
	Class Log{
		Propagation.REQUIRED_NEW  
		insertLog();  
	}
	
	Propagation.REQUIRED
	Void  saveDept(){
		insertLog();    // 始终开启事务
		.. 异常, 日志不会回滚
		saveDept();
	}








