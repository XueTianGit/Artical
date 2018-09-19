title: Hibernate-HQL查询
date: 2016/3/9 9:19:20  
categories: WebFrame
---

# Hibernate-HQL查询 #

HQL查询与SQL查询区别：

		SQL: (结构化查询语句)查询的是表以及字段;  不区分大小写。
		HQL: hibernate  query  language 即hibernate提供的面向对象的查询语言
			查询的是对象以及对象的属性。区分大小写。

> 除了 Java 类与属性的名称外，查询语句对大小写并不敏感。 所以 SeLeCT 与 sELEct 以及 SELECT 是相同的，
> 但是 org.hibernate.eg.FOO 并不等价于 org.hibernate.eg.Foo 并且 foo.barSet 也不等价于 foo.BARSET。

在使用HQL查询时，我们一般都指定 auto-import:
	`<hibernate-mapping package="com.suixin.c_hbm_config" auto-import="true">`
这样我们在写HQL查询语句，对于类就不需要写全名了。

## 基本查询 ##
	public class App {
		
		private static SessionFactory sf;
		static {
			sf = new Configuration()
				.configure()
				.addClass(Dept.class)   
				.addClass(Employee.class)   // 测试时候使用，会自动加载映射文件：Employee.hbm.xml
				.buildSessionFactory();
		}
	
		@Test
		public void all() {
		
			Session session = sf.openSession();
			session.beginTransaction();	

			//	HQL查询
			// 注意：使用hql查询的时候 auto-import="true" 要设置true， 如果是false，写hql的时候，要指定类的全名
			
			//简单的返回 Dept 类的所有实例
			Query q = session.createQuery("from Dept");
			System.out.println(q.list());
			
			// a. 查询全部列
			Query q = session.createQuery("from Dept");  //OK
		//	Query q = session.createQuery("select * from Dept");  // 错误，不支持 *
			Query q = session.createQuery("select d from Dept d");  // OK  d是别名
			System.out.println(q.list());
	
			// b. 查询指定的列  【返回对象数据Object[]】
			Query q = session.createQuery("select d.deptId,d.deptName from Dept d");  
			System.out.println(q.list());
			
			// c. 查询指定的列, 自动封装为对象  【必须要提供带参数构造器】，把对象放到Query对象的list集合中
			Query q = session.createQuery("select new Dept(d.deptId,d.deptName) from Dept d");  
			System.out.println(q.list());
			
			// d. 条件查询: 一个条件/多个条件and or/between and/模糊查询
			// 条件查询： 占位符，当让我们也刻可以不适用占位符，直接写条件
			Query q = session.createQuery("from Dept d where deptName=?");
			q.setString(0, "财务部");
			q.setParameter(0, "财务部");
			System.out.println(q.list());
			
			// 条件查询： 命名参数 代替问号呗
			Query q = session.createQuery("from Dept d where deptId=:myId or deptName=:name");
			q.setParameter("myId", 12);
			q.setParameter("name", "财务部");
			System.out.println(q.list());
			
			// 范围
			Query q = session.createQuery("from Dept d where deptId between ? and ?");
			q.setParameter(0, 1);
			q.setParameter(1, 20);
			System.out.println(q.list());
			
			// 模糊
			Query q = session.createQuery("from Dept d where deptName like ?");
			q.setString(0, "%部%");
			System.out.println(q.list());
			
	
			// e. 聚合函数统计
			Query q = session.createQuery("select count(*) from Dept");
			Long num = (Long) q.uniqueResult();
			System.out.println(num);
			
			// f. 分组查询
			//-- 统计t_employee表中，每个部门的人数
			//数据库写法：SELECT dept_id,COUNT(*) FROM t_employee GROUP BY dept_id;
			// HQL写法
			Query q = session.createQuery("select e.dept, count(*) from Employee e group by e.dept");
			System.out.println(q.list())；	
			
			session.getTransaction().commit();
			session.close();
		}
		


## 链接查询 ##

根据两个表或多个表列之间的关系，从这些表中查询数据。其实就是为了实现多表查询！！！！！！！！
由于一些原因，这里只是列出代码，日后必定搞清。。。。。

		@Test
		public void join() {
			
			Session session = sf.openSession();
			session.beginTransaction();
			
			//1) 内连接   【映射已经配置好了关系，关联的时候，直接写对象的属性即可】
	//		Query q = session.createQuery("from Dept d inner join d.emps");
			
			//2) 左外连接
	//		Query q = session.createQuery("from Dept d left join d.emps");
	
			//3) 右外连接
			Query q = session.createQuery("from Employee e right join e.dept");
			q.list();
			
			session.getTransaction().commit();
			session.close();
		}
		
		// g. 连接查询 - 迫切连接
		@Test
		public void fetch() {
			
			Session session = sf.openSession();
			session.beginTransaction();
			
			//1) 迫切内连接    【使用fetch, 会把右表的数据，填充到左表对象中！】
	//		Query q = session.createQuery("from Dept d inner join fetch d.emps");
	//		q.list();
			
			//2) 迫切左外连接
			Query q = session.createQuery("from Dept d left join fetch d.emps");
			q.list();
			
			session.getTransaction().commit();
			session.close();
		}
		
## HQL查询优化 ##
> 一般HQL查询语句，在createQuery（）方法中我们就已经把HQL语句写死了，
> 奶奶的，抱着什么都要写活得思想，我们可以把HQL语句写在配置文件中


在Dept.hbm.xml文件中，我们可以这样配置:

	<!-- 存放sql语句 -->
	<query name="getAllDept">
		<![CDATA[
			from Dept d where deptId < ?
		]]>
	</query>

代码中的使用：

		@Test
		public void hql_other() {
			Session session = sf.openSession();
			session.beginTransaction();
			// HQL写死
	//		Query q = session.createQuery("from Dept d where deptId < 10 ");
			
			// HQL 放到映射文件中
			Query q = session.getNamedQuery("getAllDept");
			q.setParameter(0, 10);
			System.out.println(q.list());
			
			session.getTransaction().commit();
			session.close();
		}
	}
		 

