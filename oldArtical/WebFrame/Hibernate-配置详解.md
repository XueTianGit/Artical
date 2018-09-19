title: Hibernate-配置详解
date: 2016/3/9 9:19:20  
categories: WebFrame
---


# Hibernate-配置详解 #

## 主配置文件 ##

- Hibernate.cfg.xml: 主配置文件中主要配置：数据库连接信息、其他参数、映射信息。
- 常用配置查看源码：hibernate-distribution-3.6.0.Final\project\etc\hibernate.properties

### 数据库链接信息 ###
例如MySQL的属性与值:

	## MySQL
	#hibernate.dialect 							->org.hibernate.dialect.MySQLDialect
	#hibernate.dialect 							->org.hibernate.dialect.MySQLInnoDBDialect
	#hibernate.dialect 							->org.hibernate.dialect.MySQLMyISAMDialect
	#hibernate.connection.driver_class 			->com.mysql.jdbc.Driver
	#hibernate.connection.url            		->jdbc:mysql:///test
	#hibernate.connection.username 				->xxx
	#hibernate.connection.password				->xxx
	
在主配置文件的体现：

	<!DOCTYPE hibernate-configuration PUBLIC
	"-//Hibernate/Hibernate Configuration DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

	<hibernate-configuration>
		<session-factory>
			<!-- 数据库连接配置 -->
			<property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
			<property name="hibernate.connection.url">jdbc:mysql:///hib_demo</property>
			<property name="hibernate.connection.username">root</property>
			<property name="hibernate.connection.password">root</property>
			<property name="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</property>
			

		</session-factory>
	</hibernate-configuration>



### 其他参数信息配置 ###

1） 自动建表
不同选值，所代表的含义：

	#hibernate.hbm2ddl.auto     create-drop     ->每次在创建sessionFactory时候执行创建表； 当调用sesisonFactory的close方法的时候，删除表！
	#hibernate.hbm2ddl.auto     create          ->每次都重新建表； 如果表已经存在就先删除再创建
	#hibernate.hbm2ddl.auto     update          ->如果表不存在就创建； 表存在就不创建；
	#hibernate.hbm2ddl.auto     validate        -> (生成环境时候) 执行验证： 当映射文件的内容与数据库表结构不一样的时候就报错！

example :

	<property name="hibernate.hbm2ddl.auto">update</property>

对于自动建表，我们也可以使用代码方式控制：

	// 自动建表
	@Test
	public void testCreate() throws Exception {
		// 创建配置管理类对象
		Configuration config = new Configuration();
		// 加载主配置文件
		config.configure();
		
		// 创建工具类对象
		SchemaExport export = new SchemaExport(config);
		// 建表
		// 第一个参数： 是否在控制台打印建表语句
		// 第二个参数： 是否执行脚本
		export.create(true, true);
	}

格式化sql

	<property name="hibernate.format_sql">true</property>

显示hibernate在运行时候执行的sql语句

	<property name="hibernate.show_sql">true</property>

### 映射信息 ###

> 我们可以将ORM配置在主配置文件中，也可以配置在其他的文件中，然后引入到主配置文件。一般都是引入。即加载映射配置：

	<mapping resource="com/suixin/a_hello/Employee.hbm.xml"/>


## 映射配置 ##
> 一般对于每一个对象，我们都要创建一个映射文件来实现ORM。

例如Employee.hbm.xml， 这个文件是对Employee对象的映射， 它的存放位置一般应于这个对象的java文件位于同一级目录。

Employee.hbm.xml配置范例：
 
	<?xml version="1.0"?>
	<!DOCTYPE hibernate-mapping PUBLIC 
	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

	<!-- 
	package: 要映射的对象所在的包(可选,如果不指定,此文件所有的类都要指定全路径)
	auto-import 默认为true， 在写hql的时候自动导入包名
				如果指定为false, 再写hql的时候必须要写上类的全名；
				如：session.createQuery("from cn.itcast.c_hbm_config.Employee").list();
 	-->
	<hibernate-mapping package="com.suixin.c_hbm_config" auto-import="true">
	
	
		<!-- 
			class 映射某一个对象的(一般情况，一个对象写一个映射文件，即一个class节点)
			name 指定要映射的对象的类型
			table 指定对象对应的表；  如果没有指定表名，默认与对象名称一样 
		 -->
		<class name="Employee" table="employee">	
			<!-- 主键 ，映射-->
			<id name="empId" column="id">
				<!-- 
					主键的生成策略
						identity   自增长(mysql,db2)
						sequence   自增长(序列)， oracle中自增长是以序列方法实现
						native     自增长【会根据底层数据库自增长的方式选择identity或sequence】
								   如果是mysql数据库, 采用的自增长方式是identity
								   如果是oracle数据库， 使用sequence序列的方式实现自增长
						
						increment  自增长(会有并发访问的问题，一般在服务器集群环境使用会存在问题。)
						
						assigned   指定主键生成策略为手动指定主键的值
						uuid       指定uuid随机生成的唯一的值
						foreign    (外键的方式)
				 -->
				<generator class="uuid"/>
			</id>
			
			<!-- 
				普通字段映射
				property
					name   指定对象的属性名称
					column 指定对象属性对应的表的字段名称，如果不写默认与对象属性一致。
					length 指定字符的长度, 默认为255
					type   指定映射表的字段的类型，如果不指定会匹配属性的类型
						   java类型：     必须写全名
						   hibernate类型：  直接写类型，都是小写
			-->
	
			<property name="empName" column="empName" type="java.lang.String" length="20"></property>
			<property name="workDate" type="java.util.Date"></property>
			<!-- 如果列名称为数据库关键字，需要用反引号或改列名。 -->
			<property name="desc" column="`desc`" type="java.lang.String"></property>
			
		</class>

    </hibernate-mapping>
	 
### 复合主键映射 ###

在上面我们配置主键时，还可以将多列联合，从而构成主键。
范例：

 
// 复合主键类

	public class CompositeKeys implements Serializable{
		private String userName;
		private String address;
	   // .. get/set
	}	   
	public class User {
	
		// 名字跟地址，不会重复
		private CompositeKeys keys;
		private int age;
	}	   

	//User.hbm.xml	   
	<?xml version="1.0"?>
	<!DOCTYPE hibernate-mapping PUBLIC 
	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

	<hibernate-mapping package="cn.itcast.d_compositeKey" auto-import="true">	
		<class name="User">
			
			<!-- 复合主键映射 -->
			<composite-id name="keys">
				<key-property name="userName" type="string"></key-property>
				<key-property name="address" type="string"></key-property>
			</composite-id>
			
			<property name="age" type="int"></property>		
		</class>
	</hibernate-mapping>
	   
	//App.java	   

	public class App2 {
		
		private static SessionFactory sf;
		static  {		
			// 创建sf对象
			sf = new Configuration()
				.configure()
				.addClass(User.class)  //（测试） 会自动加载映射文件：Employee.hbm.xml
				.buildSessionFactory();
		}
	
		//1. 保存对象
		@Test
		public void testSave() throws Exception {
			Session session = sf.openSession();
			Transaction tx = session.beginTransaction();
			
			// 对象
			CompositeKeys keys = new CompositeKeys();
			keys.setAddress("广州棠东");
			keys.setUserName("Jack");
			User user = new User();
			user.setAge(20);
			user.setKeys(keys);
			
			// 保存
			session.save(user)
			tx.commit();
			session.close();
		}
		
		@Test
		public void testGet() throws Exception {
			Session session = sf.openSession();
			Transaction tx = session.beginTransaction();
			
			//构建主键再查询
			CompositeKeys keys = new CompositeKeys();
			keys.setAddress("广州棠东");
			keys.setUserName("Jack");
			
			// 主键查询
			User user = (User) session.get(User.class, keys);
			// 测试输出
			if (user != null){
				System.out.println(user.getKeys().getUserName());  //**
				System.out.println(user.getKeys().getAddress());   //**
				System.out.println(user.getAge());
			}
			
			
			tx.commit();
			session.close();
		}
	}
	 
