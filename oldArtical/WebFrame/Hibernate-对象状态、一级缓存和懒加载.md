title: Hibernate-对象状态、一级缓存和懒加载
date: 2016/3/9 9:19:20  
categories: WebFrame
---

# Hibernate-对象状态、一级缓存和懒加载 #


## 对象状态 ##

例如对于： User user = new User();

user有这几个状态：

### 临时状态 ###
> 特点： 直接new出来的对象;  不处于session的管理; 数据库中没有对象的记录;

### 持久化状态 ###
> - 当调用session的save/saveOrUpdate/getloa/d/list等方法的时候，对象就是持久化状态。
> - 处于持久化状态的对象，当对对象属性进行更改的时候，会反映到数据库中!
> - 特点： 处于session的管理; 数据库中有对应的记录;、

### 游离状态 ###
> - 它是Session关闭后，对象的状态；
> - 特点 ：
>     - 不处于session的管理；
>     - 数据库中有对应的记录

![](http://7xrbxa.com1.z0.glb.clouddn.com/WebFrameHibernate-%E5%AF%B9%E8%B1%A1%E7%8A%B6%E6%80%81%E5%9B%BE.png)

代码举例：

	public void testSaveSet() throws Exception {
		Session session = sf.openSession();
		session.beginTransaction();
		
		// 创建对象						【临时状态】
		User user = new User();
		user.setUserName("Jack22222");
		// 保存							【持久化状态】
		session.save(user);		
		user.setUserName("Jack333333");  // 会反映到数据库	
		
		// 查询
		User user = (User) session.get(User.class, 5);
		user.setUserName("Tomcat");// hibernate会自动与数据库匹配（一级缓存），如果不一样就更新数据库	
	
		session.getTransaction().commit();
		session.close();

		user.setUserName("Jack444444444");  //不会更新到数据库
		// 打印							【游离状态】
		System.out.println(user.getUserId());
		System.out.println(user.getUserName());
	}
	

## 一级缓存 ##

> 使用一级缓存可以减少向数据库发送HQL语句的数量，从而减小数据库的压力，也避免不必要的查询。

### 概念 ###

> - Hibenate中一级缓存，也叫做session的缓存，它可以在session范围内减少数据库的访问次数！只在session范围有效！Session关闭，一级缓存失效！
> - 当调用session的save/saveOrUpdate/get/load/list/iterator方法的时候，都会把对象放入session的缓存中。 
> - Session的缓存由hibernate维护， 用户不能操作缓存内容； 如果想操作缓存内容，必须通过hibernate提供的evit/clear方法操作。
>- 特点：
>   - 只在(当前)session范围有效，作用时间短，效果不是特别明显！
>   - 在短时间内多次操作数据库，效果比较明显！

->缓存相关几个方法的作用

	session.flush();        让一级缓存与数据库同步
	session.evict(arg0);    清空一级缓存中指定的对象
	session.clear();        清空一级缓存中缓存的所有对象


	NT:
	1)不同的session是不会共享缓存数据。
	
	2） list与iterator查询的区别
	list() 
	一次把所有的记录都查询出来，
	会放入缓存，但不会从缓存中获取数据
	Iterator()
	N+1查询； N表示所有的记录总数
	即会先发送一条语句查询所有记录的主键（1），
	再根据每一个主键再去数据库查询（N）！
	会放入缓存，也会从缓存中取数据！


->session的缓存举例

> 当用户需要对指定的对象进行修改的时候，如果对于同一个属性修改了多次，其实hibernate的session缓存并不是执行多个update语句，而是以最后一个更新为准而发送一条更新的sql。

代码举例：

	//查询
	user = (User) session.get(User.class, 5);// 先检查缓存中是否有数据，如果有不查询数据库，直接从缓存中获取   -> 如果这里有，就发送HQL
	user = (User) session.get(User.class, 5);// 先检查缓存中是否有数据，如果有不查询数据库，直接从缓存中获取   ->上面已经干完了。不会再发送了
	//不同的session缓存各不相关
		// user放入session1的缓存区
	User user = (User) session1.get(User.class, 1);
		// user放入session2的缓存区
	session2.update(user);


## 懒加载 ##
> 我们所说的懒加载也被称为延迟加载，它在查询的时候不会立刻访问数据库，而是返回代理对象，当真正去使用对象的时候才会访问数据库。

实现懒加载的前提: 
> - 实体类不能是final的
> - 能实现懒加载的对象都是被CGLIB（反射调用）改写的代理对象,所以不能是final修饰的
> - 须要asm,cglib两个jar包
> - 相应的lazy属性为true
> - 相应的fetch属性为select 

问题：
Hibernate中 get()方法与 load()方法的区别是什么？
> get: 及时加载，只要调用get方法立刻向数据库查询
> load:默认使用懒加载，当用到数据的时候才向数据库查询。 

**在真正使用数据的时候才向数据库发送查询的sql；例如，如果调用集合的size()/isEmpty()方法，只是统计，不真正查询数据！**

	对于load方法：
	// load，默认懒加载， 及在使用数据的时候，才向数据库发送查询的sql语句！
	dept = (Dept)session.load(Dept.class, 9);   //即这里不会向数据库发送HQL语句

### 懒加载功能实现 ###
这段参考其他博主文章（不过我也进行了辛苦的排版。。。。。）：

> 通过Session.load()实现懒加载

	load(Object, Serializable)：根据id查询 。查询返回的是代理对象，不会立刻访问数据库，是懒加载的。当真正去使用对象的时候才会访问数据库。
	用load()的时候会发现不会打印出查询语句，而使用get()的时候会打印出查询语句。
	使用load()时如果在session关闭之后再查询此对象，会报异常：could not initialize proxy - no Session。处理办法：在session关闭之前初始化一下查询出来的对象：Hibernate.initialize(user);
	使用load()可以提高效率，因为刚开始的时候并没有查询数据库。但很少使用。

> one-to-one(元素)实现了懒加载。
 
	在一对一的时候，查询主对象时默认不是懒加载。即：查询主对象的时候也会把从对象查询出来。
	要把主对象配制成lazy="true" constrained="true"  fetch="select"。此时查询主对象的时候就不会查询从对象，从而实现了懒加载。
	一对一的时候，查询从对象的是默认是懒加载。即：查询从对象的时候不会把主对象查询出来。而是查询出来的是主对象的代理对象。

> many-to-one（元素）实现了懒加载。

	多对一的时候，查询主对象时默认是懒加载。即：查询主对象的时候不会把从对象查询出来。
	多对一的时候，查询从对象时默认是懒加载。即：查询从对象的时候不会把主对象查询出来。
	hibernate3.0中lazy有三个值，true，false，proxy,默认的是lazy="proxy".具体设置成什么要看你的需求，并不是说哪个设置就是最好的。在<many-to-one>与<one-to-one>标签上:当为true时,会有懒加载特性,当为false时会产生N+1问题,比如一个学生对应一个班级,用一条SQL查出10个学生,当访问学生的班级属性时Hibernate会再产生10条SQL分别查出每个学生对应的班级.
	lazy= 什么时候捉取
	fetch= 捉取方式:select=关联查询;join=连接表的方式查询(效率高)
	fetch=join时,lazy的设置将没有意义.

> one-to-many(元素)懒加载：默认会懒加载，这是必须的，是重常用的。

	一对多的时候，查询主对象时默认是懒加载。即：查询主对象的时候不会把从对象查询出来。
	一对多的时候，查询从对象时默认是懒加载。即：查询从对象的时候不会把主对象查询出来。
	需要配置主对象中的set集合lazy="false" 这样就配置成是不懒加载了。或者配置抓取方式fetch="join"也可以变成不懒加载。 

> 实现懒加载的方案:
 
	方法一:(没有使用懒加载)  
	用 Hibernate.initialize(de.getEmps()) 提前加载一下. 
	方法二:
	把与Session脱离的对象重新绑定
	lock()方法是用来让应用程序把一个未修改的对象重新关联到新session的方法。
	//直接重新关联
	session.lock(fritz,LockMode.NONE);
	//进行版本检查后关联
	session.lock(izi,LockMode.READ);
	//使用SELECT... FOR UPDATE进行版本检查后关联
	session.lock(pk,LockMode.UPGRADE);
	方法三:
	OpenSessionInVie
	参见 http://www.javaeye.com/topic/32001    
     
### fetch 和 lazy 配置用于数据的查询  ###

	lazy 参数值常见有 false 和 true，Hibernate3映射文件中默认lazy = true ；
	fetch 指定了关联对象抓取的方式，参数值常见是select和join，默认是select,select方式先查询主对象，再根据关联外键，每一个对象发一个select查询，获取关联的对象，形成了n+1次查询；而join方式，是leftouter join查询，主对象和关联对象用一句外键关联的sql同时查询出来，不会形成多次查询。 
	在映射文件中，不同的组合会使用不同的查询： 
	1、lazy="true" fetch = "select" ，使用延迟策略，开始只查询出主对象，关联对象不会查询，只有当用到的时候才会发出sql语句去查询 ；
	2、lazy="false" fetch = "select" ，没有用延迟策略，同时查询出主对象和关联对象，产生1+n条sql. 
	3、lazy="true"或lazy="false"fetch = "join"，延迟都不会作用，因为采用的是外连接查询，同时把主对象和关联对象都查询出来了. 
	另 外，在hql查询中,配置文件中设置的join方式是不起作用的,而在其他查询方式如get、criteria等是有效的，使用 select方式;除非在hql中指定join fetch某个关联对象。fetch策略用于get/load一个对象时，如何获取非lazy的对象/集合。 这些参数在Query中无效。

### 懒加载异常 ###
	Session关闭后，不能使用懒加载数据！
	如果session关闭后，使用懒加载数据报错：
	org.hibernate.LazyInitializationException: could not initialize proxy - no Session
	
	如何解决呢？
	方式1： 先使用一下数据，再关闭session
	方式2：强迫代理对象初始化，(load方法返回的是所需对象的代理对象）   Hibernate.initialize(dept);
	方式3：关闭懒加载设置lazy=false;
	方式4： 在使用数据之后，再关闭session！ 




