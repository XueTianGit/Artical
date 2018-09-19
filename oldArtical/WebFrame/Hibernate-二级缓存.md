title: Hibernate-二级缓存
date: 2016/3/9 9:19:20  
categories: WebFrame
---

# Hibernate-二级缓存 #

## 简述 ##

> - Hibernate提供了基于应用程序级别的缓存（作用在SessionFactory范围内的）， 可以跨多个session，即不同的session都可以访问缓存数据。 这个缓存也叫二级缓存。
> - Hibernate提供的二级缓存有默认的实现，且是一种可插配的缓存框架！如果用户想用二级缓存，只需要在hibernate.cfg.xml中配置即可； 不想用，直接移除，不影响代码。
> - 如果用户觉得hibernate提供的框架框架不好用，自己可以换其他的缓存框架或自己实现缓存框架都可以。

二级缓存的工作可以概括为以下几个部分：

- 在执行各种条件查询时，会把查询到的对象放入到缓存中
- 当Hibernate访问数据对象的时候，首先会从Session一级缓存中查找，如果查不到并且配置了二级缓存，那么会从二级缓存中查找，如果还查不到，就会查询数据库，并把数据缓存。
- 删除、更新、增加数据的时候，同时更新缓存。


## 二级缓存的配置 ##

在hibernate.properties配置文件， 我们可以看到二级缓存的配置选项：

	##########################
	### Second-level Cache ###
	##########################
	
	#hibernate.cache.use_second_level_cache false【二级缓存默认不开启，需要手动开启】
	#hibernate.cache.use_query_cache true  【开启查询缓存】
	
	## choose a cache implementation		【二级缓存框架的实现】
	
	#hibernate.cache.provider_class org.hibernate.cache.EhCacheProvider
	#hibernate.cache.provider_class org.hibernate.cache.EmptyCacheProvider
	hibernate.cache.provider_class org.hibernate.cache.HashtableCacheProvider   ->   默认实现
	#hibernate.cache.provider_class org.hibernate.cache.TreeCacheProvider
	#hibernate.cache.provider_class org.hibernate.cache.OSCacheProvider
	#hibernate.cache.provider_class org.hibernate.cache.SwarmCacheProvider

## 二级缓存的简单使用步骤 ##

> 开启二级缓存,在hibernate.cfg.xml文件中：

	<property name="hibernate.cache.use_second_level_cache">true</property>

> 指定缓存框架

	<property name="hibernate.cache.provider_class">org.hibernate.cache.HashtableCacheProvider</property>

> 指定将哪些类放入到二级缓存

	<class-cache usage="read-only" class="com.suixin.second_cache.Employee"/>	   //对普通对象的缓存

###  缓存策略  ###
> 即上面的usage属性：

	<class-cache usage="read-only"/>              放入二级缓存的对象，只读; 
	<class-cache usage="nonstrict-read-write"/>   非严格的读写
	<class-cache usage="read-write"/>   			  读写； 放入二级缓存的对象可以读、写；
	<class-cache usage="transactional"/>          (基于事务的策略)


## 集合缓存 ##

> 对于缓存的内容，我们可以指定到一个集合对象，当在缓存集合对象是，集合中的元素当然也会放到缓存中
	
	例如：
	在Dept对象中定义着这样一个集合：
		private Set<Employee> emps = new HashSet<Employee>();
	
	缓存他：
		<!-- 集合缓存[集合缓存的元素对象，也加加入二级缓存] -->
		<collection-cache usage="read-write" collection="com.suixin.second_cache.Dept.emps"/>

##  setCacheable() ##

	使用这个方法可以： 指定从二级缓存找，或者是放入二级缓存
	Query q = session1.createQuery("from Dept").setCacheable(true);

## 查询缓存 ##

> - hibernate的查询缓存是主要是针对普通属性结果集的缓存， 而对于实体对象的结果集只缓存id。
> - 在一级缓存,二级缓存和查询缓存都打开的情况下作查询操作时这样的：查询普通属性，会先到查询缓存中取，
> - 如果没有，则查询数据库；查询实体，会先到查询缓存中取id，
> - 如果有，则根据id到缓存(一级/二级)中取实体，如果缓存中取不到实体，再查询数据库。（二级缓存开启时）
> 和一级/二级缓存不同，查询缓存的生命周期 ，是不确定的，当前关联的表发生改变时，查询缓存的生命周期结束。
 
	查询缓存的配置和使用也是很简单的：
	1>查询缓存的启用不但要在配置文件中进行配置
	   <property name="hibernate.cache.use_query_cache">true</property>
	2>还要在程序中显示的进行启用
	    query.setCacheable(true);
	
	list() 默认情况只会放入缓存，不会从缓存中取！
	而使用查询缓存，可以让他去从二级缓存中取！！

## 二级缓存和Hibernate查询缓存结合使用 ##

当只是用Hibernate查询缓存而关闭二级缓存的时候： 

- 如果查询的是部分属性结果集： 那么当第二次查询的时候就不会发出SQL，直接从Hibernate查询缓存中取数据；
- 如果查询的是实体结果集eg(from Student) ，首先Hibernate查询缓存存放实体的ID，第二次查询的时候就到Hibernate查询缓存中取出ID 
    一条一条的到数据库查询，这样，将发出N 条SQL造成了SQL泛滥。

当都开启Hibernate查询缓存和二级缓存的时候：

- 如果查询的是部分属性结果集： 这个和上面只是用Hibernate查询缓存而关闭 二级缓存的时候一致，因为不涉及实体不会用到二级缓存；
- 如果查询的是实体结果集eg(from Student)，首先Hibernate查询缓存存放实体的ID，第二次查询的时候，
   就到Hibernate查询缓存中取出ID，到二级缓存区找数据，如果有数据，就不会发出SQL；如果都有，一条SQL都不会发出，直接从二级缓存中取数据。


















