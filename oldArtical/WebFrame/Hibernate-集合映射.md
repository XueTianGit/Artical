title: Hibernate-映射
date: 2016/3/9 9:19:20  
categories: WebFrame
---

# Hibernate-映射 #


## 集合映射 ##
> 集合映射是关联映射的基础。

以这个javabean为例，分别进行Set、List、Map的集合映射。

	public class User {
	
		private int userId;
		private String userName;
		// 一个用户，对应的多个地址
		private Set<String> address;
		private List<String> addressList = new ArrayList<String>(); 
		//private String[] addressArray; // 映射方式和list一样     <array name=""></array>
		private Map<String,String> addressMap = new HashMap<String, String>();
		
	}

	<class name="User" table="t_user">
		<id name="userId" column="id">
			<generator class="native"></generator>
		</id>	
		<property name="userName"></property>
	</class>

###  Set集合映射  ###

> 集合映射都是差不多的，基本上都对应表的一对多。在多的一方都需要添加外键。
	
		<!-- 
			set集合属性的映射
				name 指定要映射的set集合的属性
				table 集合属性要映射到的表
				key  指定集合表(t_address)的外键字段(一对多)
				element 指定集合表的其他字段
					type 元素类型，一定要指定
		 -->
		 <set name="address" table="t_address">
		 	<key column="uid"></key>
		 	<element column="address" type="string"></element>
		 </set>

###  List集合映射 ###
		 
		 <!-- 
		 	list集合映射
		 		list-index  指定的是排序列的名称 (因为要保证list集合的有序)
		  -->
		  <list name="addressList" table="t_addressList">
		  	  <key column="uid"></key>
		  	  <list-index column="idx"></list-index>
		  	  <element column="address" type="string"></element>
		  </list>

### Map集合映射 		  ### 
		  <!-- 
		  	map集合的映射
		  		key  指定外键字段
		  		map-key 指定map的key 
		  		element  指定map的value
		   -->
		  <map name="addressMap" table="t_addressMap">
		  	<key column="uid"></key>
		  	<map-key column="shortName" type="string" ></map-key>
		  	<element column="address" type="string" ></element>
		  </map>


## 关联映射 ##

### 多对一映射与一对多 ###

> 即在多的一方加外键。注意下面配置中外键字段一定要相同！！

需要进行映射的javabean

	public class Dept {
		private int deptId;
		private String deptName;
		// 【一对多】 部门对应的多个员工
		private Set<Employee> emps = new HashSet<Employee>();
		
	public class Employee {
	
		private int empId;
		private String empName;
		private double salary;
		// 【多对一】员工与部门
		private Dept dept;	


	<hibernate-mapping package="com.suixin.b_one2Many">
	
		<class name="Dept" table="t_dept">
			<id name="deptId">
				<generator class="native"></generator>
			</id>	
			<property name="deptName" length="20"></property>
			
			<!-- 
				一对多关联映射配置  （通过部门管理到员工）
				Dept 映射关键点：
				1.  指定 映射的集合属性： "emps"
				2.  集合属性对应的集合表： "t_employee"
				3.  集合表的外键字段   "t_employee. dept_id"（必须和Employee映射时的外键字段相同）
				4.  集合元素的类型
				
			 -->
			 <set name="emps">   <!-- table="t_employee" -->
			 	 <key column="dept_id"></key>
			 	 <one-to-many class="Employee"/>
			 </set>
		</class>
		
	</hibernate-mapping>


	<hibernate-mapping package="com.suixin.b_one2Many">
		<class name="Employee" table="t_employee">
			<id name="empId">
				<generator class="native"></generator>
			</id>	
			<property name="empName" length="20"></property>
			<property name="salary" type="double"></property>
			
			<!-- 
				多对一映射配置
				Employee 映射关键点：
				1.  映射的部门属性  ：  dept
				2.  映射的部门属性，对应的外键字段: dept_id
				3.  部门的类型
				这个<mang-to-one>配置的其实就是Employee的外键字段
			 -->
			 <many-to-one name="dept" column="dept_id" class="Dept"></many-to-one>	 
		</class>
	</hibernate-mapping>

	 
对于这种映射， Hibernate是如何建表和进行保存的呢？

	public class App {	
		private static SessionFactory sf;
		static {
			sf = new Configuration()
				.configure()
				.addClass(Dept.class)   
				.addClass(Employee.class)   // 测试时候使用  
				.buildSessionFactory();
		}
	
		// 保存， 部门方 【一的一方法操作】
		@Test
		public void save() {
			
			Session session = sf.openSession();
			session.beginTransaction();
			
			// 部门对象
			Dept dept = new Dept();
			dept.setDeptName("应用开发部");
			// 员工对象
			Employee emp_zs = new Employee();
			emp_zs.setEmpName("张三");
			Employee emp_ls = new Employee();
			emp_ls.setEmpName("李四");
			// 关系
			dept.getEmps().add(emp_zs);
			dept.getEmps().add(emp_ls);
	
			// 保存
			session.save(emp_zs);
			session.save(emp_ls);
			session.save(dept); // 保存部门，部门下所有的员工  
			
			session.getTransaction().commit();
			session.close();
			/*
			 *  结果
			 *  Hibernate: insert into t_employee (empName, salary, dept_id) values (?, ?, ?)
				Hibernate: insert into t_employee (empName, salary, dept_id) values (?, ?, ?)
				Hibernate: insert into t_dept (deptName) values (?)
				Hibernate: update t_employee set deptId=? where empId=?    维护员工引用的部门的id
				Hibernate: update t_employee set deptId=? where empId=?
			 */
		}
		// 【推荐】 保存， 员工方 【多的一方法操作】
		@Test
		public void save2() {
			
			Session session = sf.openSession();
			session.beginTransaction();
			
			// 部门对象
			Dept dept = new Dept();
			dept.setDeptName("综合部");
			// 员工对象
			Employee emp_zs = new Employee();
			emp_zs.setEmpName("张三");
			Employee emp_ls = new Employee();
			emp_ls.setEmpName("李四");
			// 关系
			emp_zs.setDept(dept);
			emp_ls.setDept(dept);
			
			
			// 保存
			session.save(dept);    // 先保存一的方法
			session.save(emp_zs);
			session.save(emp_ls);  // 再保存多的一方，关系回自动维护(映射配置完)
			
			session.getTransaction().commit();
			session.close();
			/*
			 *  结果
			 *  Hibernate: insert into t_dept (deptName) values (?)
				Hibernate: insert into t_employee (empName, salary, dept_id) values (?, ?, ?)
				Hibernate: insert into t_employee (empName, salary, dept_id) values (?, ?, ?)
				少生成2条update  sql
			 */
		}
		
	}


#### Inverse属性 ####

- Inverse属性，是在维护关联关系的时候起作用的。表示控制权是否反转。（relationship_owner）
- inverse"关键字在一对多关系和多对多关系中被声明使用（在多对一中没有inverse关键字）。
它的取值决定了具有关联关系（一对多或多对多）的两个实体类哪一个负责维护二者之间的关系。
- 总之，inverse="true"说明当前配置文件对应的实体类不是关系的拥有者，而其对应的关联实体类是关系的拥有者即负责维护关系

> 维护关系，即是否维护外键字段

	Inverse , 控制反转：
	Inverse = false  不反转（默认）；   **当前方有控制权**
			= True  控制反转；   ** 当前方没有控制权**


	对于inverse的配置，会影响Hibernate的HQL语句的生成数量：
	就向上面的代码：
	在配置<one-to-many>时，由于没有指定inverse属性，默认为false，即控制不反转，Dept具有维护关系的责任。
	所以上面测试代码中这些语句执行时：

		dept.getEmps().add(emp_zs);
		dept.getEmps().add(emp_ls);

		// 保存
		session.save(emp_zs);
		session.save(emp_ls);
		session.save(dept); // 保存部门，部门下所有的员工  

> 总共生成了5条HQL语句。对于关系的维护体现在保存部门时，最后的两条update语句。
> 即部门去维护了与员工的关系。


而如果这样保存：

		session.save(dept);    // 先保存一的方法
		session.save(emp_zs);
		session.save(emp_ls);  // 再保存多的一方

就少生成了两条update语句，（这两条语句也没必要生成，请看上面代码）。

> 所以（在没有去配置inverse属性时）：
> 在一对多与多对一的关联关系中，保存数据最好的通过多的一方来维护关系，这样可以减少update语句的生成，从而提高hibernate的执行效率！


就向在上例维护关联关系中，是否设置inverse属性：

	1. 保存数据 -> 有影响。
	   如果设置控制反转,即inverse=true, 然后通过部门方（上例）维护关联关系。在保存部门的时候，同时保存员工， 数据会保存，但关联关系不会维护。即外键字段为NULL。
	3. 解除关联关系 -> 有影响。
	   inverse=false，  可以解除关联
	   inverse=true，   当前方(部门)没有控制权，不能解除关联关系（）
	4. 删除数据对关联关系的影响 -> 有影响。
		inverse=false, 有控制权， 可以删除。先清空外键引用，再删除数据。(即我们可以直接去删除部门)
		inverse=true,  没有控制权: 如果删除的记录有被外键引用，会报错，违反主外键引用约束！  如果删除的记录没有被引用，可以直接删除。（我们不可以直接去删除部门）
#### cascade 属性 ####

cascade  表示级联操作  【可以设置到一的一方或多的一方】
> - none            不级联操作， 默认值
> - save-update     级联保存或更新
> - delete		    级联删除
> - save-update,delete       级联保存、更新、删除
> - all                      同上。级联保存、更新、删除

向上例的代码，我们把员工添加到部门后，如果设置了级联保存，那么就会自动保存员工。（很方便）

	dept.getEmps().add(emp_zs);
	dept.getEmps().add(emp_ls)；
	session.save(dept);       // 保存部门，部门下所有的员工  

------->这两个属性真的难，不想再探讨了。。。。。。。。。

### 多对多映射 ###
生成中间表保存关系。

例子：项目与开发人员

	javabean：
	public class Developer {
		private int d_id;
		private String d_name;
		// 开发人员，参数的多个项目
		private Set<Project> projects = new HashSet<Project>();
	}
	
	
	public class Project {
		private int prj_id;
		private String prj_name;
		// 项目下的多个员工
		private Set<Developer> developers = new HashSet<Developer>();
	}

映射文件：

	<hibernate-mapping package="com.suixin.c_many2many">
		
		<class name="Project" table="t_project">
			<id name="prj_id">
				<generator class="native"></generator>
			</id>	
			<property name="prj_name" length="20"></property>
			<!-- 
				多对多映射:
				1.  映射的集合属性： “developers”
				2.  集合属性，对应的中间表： “t_relation”
				3. 外键字段:  prjId
				4. 外键字段，对应的中间表字段:  did
				5.   集合属性元素的类型
			 -->
			 <set name="developers" table="t_relation" cascade="save-update">
			 	<key column="prjId"></key>
			 	<many-to-many column="did" class="Developer"></many-to-many>
			 </set>
			 
		</class>
		
	
	</hibernate-mapping>
	
	
	Developer.hbm.xml
	<?xml version="1.0"?>
	<!DOCTYPE hibernate-mapping PUBLIC 
		"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
		"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
	
	<hibernate-mapping package="com.suixin.c_many2many">
		
		<class name="Developer" table="t_developer">
			<id name="d_id">
				<generator class="native"></generator>
			</id>	
			<property name="d_name" length="20"></property>
			
			<!-- 
				多对多映射配置： 员工方
					name  指定映射的集合属性
					table 集合属性对应的中间表
					key   指定中间表的外键字段(引用当前表t_developer主键的外键字段)
					many-to-many
						column 指定外键字段对应的项目字段
						class  集合元素的类型
			 -->
			<set name="projects" table="t_relation">
				<key column="did"></key>
				<many-to-many column="prjId" class="Project"></many-to-many>
			</set>
			
			 
		</class>
	</hibernate-mapping>


#### inverse属性对多对多的影响 ####

>- 保存数据-> 有影响。

	 inverse=false ，有控制权，可以维护关联关系； 保存数据的时候会把对象关系插入中间表；
	 inverse=true,  没有控制权， 不会往中间表插入数据。
>- 获取数据->无。
>- 解除关系

	// inverse=false 有控制权， 解除关系就是删除中间表的数据。
	// inverse=true  没有控制权，不能解除关系。
>- 删除数据 ->有影响。

	// inverse=false, 有控制权。 先删除中间表数据，再删除自身。
	// inverse=true, 没有控制权。 如果删除的数据有被引用，会报错！ 否则，才可以删除


	
