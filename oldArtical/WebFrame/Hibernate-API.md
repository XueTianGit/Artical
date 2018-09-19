title: Hibernate-AP
date: 2016/3/9 9:19:20  
categories: WebFrame
---

# Hibernate-API #

## Configuration   ##

该类是配置管理类。顾名思义是用它来管理配置。

### method: ###

    config.configure();   							  
    加载主配置文件(hibernate.cfg.xml);默认加载src/hibernate.cfg.xml
	config.configure(“cn/config/hibernate.cfg.xml”);   
    加载指定路径下指定名称的主配置文件
	config.buildSessionFactory();   				  
    创建session的工厂对象


## SessionFactory ##
session的工厂（或者说它代表了这个hibernate.cfg.xml配置文件)

### method: ###
    sf.openSession();   创建一个sesison对象

    sf.getCurrentSession();  
    若Session没有创建，则创建Session，并将session绑定到当前线程上，线程结束，session会自动关闭。
    若当前线程已经绑定session，则直接获取

## Session ##
session对象维护了一个连接(Connection), 代表了与数据库连接的会话。
它是Hibernate最重要的对象，只用使用hibernate与数据库操作，都用到这个对象

### method: ###
	session.beginTransaction(); 
	 开启一个事务； **hibernate要求所有的与数据库的操作必须有事务的环境，否则报错！**
	
	有关更新的方法：
	session.save(obj);   
		 保存一个对象
	session.update(emp);  
		更新一个对象（更新是必须已经设置了主键）
	session.saveOrUpdate(emp);      
		保存或者更新： 没有设置主键，执行保存；
		有设置主键，执行更新操作; 
		如果设置主键不存在报错！

	主键查询相关的方法：
	session.get(Employee.class, 1);    
		主键查询
	session.load(Employee.class, 1);   主
		键查询 (支持懒加载)
	delete(Object object) 
	    删除一个对象
	。。。。方法太多，这里只列举一下一些常用的

#### HQL查询 ####
查看  Hibernate-HQL查询   文档。

#### Criteria查询 ####
应用并不是很广泛，我只做了简单了解。
这种查询是完全面向对象的查询

	@Test
	public void criteria() {
		
		Session session = sf.openSession();
		session.beginTransaction();
		
		Criteria criteria = session.createCriteria(Employee.class);
		// 构建条件
		criteria.add(Restrictions.eq("empId", 12));
		criteria.add(Restrictions.idEq(12));  // 主键查询
		
		System.out.println(criteria.list());
		
		session.getTransaction().commit();
		session.close();
	}	 


#### 本地SQL查询 ####
	复杂的查询，就要使用原生态的sql查询，也可以，就是本地sql查询的支持！
	(缺点： 不能跨数据库平台！)

	@Test
	public void sql() {

		Session session = sf.openSession();
		session.beginTransaction();
		
		SQLQuery q = session.createSQLQuery("SELECT * FROM t_Dept limit 5;").addEntity(Dept.class);  // 也可以自动封装
		System.out.println(q.list());
		
		session.getTransaction().commit();
		session.close();
	}

## Query ##
代表查询。。。。。。

	list() 
	将查询结果，以list形式列出
    uniqueResult() 
	返回一个唯一的实例。	
    还有一些设置查询参数的方法
    比如 setSting(),setBinary()什么的

## Transaction ##
就那些与事务相关的的方法。。。。。

	begin() 
	commit() 
	rollback()
	
	isActive()    Is this transaction still active?
	。。。。。。。

写到这里，我发现是列举不详细的，只能列举常用的，想看还是去看文档！！！！！！所以不写了。。。。。。


     