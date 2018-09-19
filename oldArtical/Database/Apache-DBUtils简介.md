title: Apache-DBUtils工具简介
date: 2016/4/5 8:45:55  
categories: Database
---

# Apache-DBUtils工具简介 #

> 作用：简化jdbc编码的工作量， 类似于Hibernate

使用这个DbUtils的一些优势：

- 防止了资源的泄露，写一段JDBC的准备代码其实并不麻烦，但是那些操作确实是十分耗时和繁琐的，也会导致有时候数据库连接忘记关闭了导致异常难以追踪。
- 干净整洁的持久化代码，把数据持久化到数据库的代码被打打削减，剩下的代码能够清晰简洁的表达你的操作目的。
- 自动把ResultSets中的工具映射到JavaBean中，你不需要手动的使用Setter方法将列值一个个赋予相应的时日，Resultset中的每一个行都大表一个完成的Bean实体。

## 使用核心 ##
* 核心类：QueryRunner ； 该类提供了query()和update()方法的多种重载形式。
-->对于这两种方法，存在接收Connection参数的形式，这是为了支持事务, 对于不接收Connection的形式，会自动在链接上发送sql，**并提交事务后，将链接返回给数据源**
* QueryRunner类，还提供了batch()方法，
* 提供很多ResultSetHandler的实现类


## 如何使用 ##
> 创建QueryRunner对象

	QueryRunner runner = new QueryRunner(JdbcUtils.getDataSource());
	NT：参数接收的是数据源，以便QueryRunner得到链接， 这里直接贴出了工具类，数据源可以使用dbcp、c3p0什么的都可以

> 执行需要的动作（CRUD）

 	@Test
	public void insert() throws SQLException{
		QueryRunner runner = new QueryRunner(JdbcUtils.getDataSource());
		String sql = "insert into users(id,name,password,email,birthday) values(?,?,?,?,?)";
		Object params[] = {2,"bbb","123","aa@sina.com",new Date()};
		runner.update(sql, params);
	}
	
	@Test
	public void update() throws SQLException{
		QueryRunner runner = new QueryRunner(JdbcUtils.getDataSource());
		String sql = "update users set email=? where id=?";
		Object params[] = {"aaaaaa@sina.com",1};
		runner.update(sql, params);
	}
	
	@Test
	public void delete() throws SQLException{
		QueryRunner runner = new QueryRunner(JdbcUtils.getDataSource());
		String sql = "delete from users where id=?";
		runner.update(sql, 1);
	}

	@Test
	public void batch() throws SQLException{
		QueryRunner runner = new QueryRunner(JdbcUtils.getDataSource());
		String sql =  "insert into users(id,name,password,email,birthday) values(?,?,?,?,?)";
		Object params[][] = new Object[3][5];
		for(int i=0;i<params.length;i++){  //3
			params[i] = new Object[]{i+1,"aa"+i,"123",i + "@sina.com",new Date()};
		}
		runner.batch(sql, params);
	}

> 对于查询，那就是重头戏了：
> 
> - 从数据库中查询数据，我们当然会得到查询结果啦。
> - 为了按我们的需求获得我们的查询结果，DBUtils框架为我们提供了接口：

	例如这个方法： Object	query(String sql, Object[] params, ResultSetHandler rsh) 
	ResultSetHandler就是一个接口， 我们可以实现这个接口，并复写 Object handle(ResultSet rs) 方法， 按照我们自己的需求处理结果。

> 不过我们自己写那多麻烦，DBUtils为我们提供一些ResultSetHandler的实现类，这些类实现了将结果封装到bean、集合等。

	例如 BeanHandler：
	@Test
	//将查询结果，封装到bean中，我们只需要将bean的class传给Handler即可
	public void find() throws SQLException{
		QueryRunner runner = new QueryRunner(JdbcUtils.getDataSource());
		String sql = "select * from users where id=?";
		User user = (User) runner.query(sql, 1, new BeanHandler(User.class));
		System.out.println(user.getEmail());
	}

> 其他的ResultSetHandler还有：

	ArrayHandler、 ArrayListHandler、BeanListHandler、
	ColumnListHandler：将结果集中某一列的数据封装到list中
	KeyedHandler： 将结果集中的每一行数据都封装到一个Map里，再把这些数据存到一个map里，其key为指定的key
	MapHandler：将结果中的第一行数据封装到一个mapli，key是列名，value是值。
	MapListHandler
	ScalarHandler:将某一列数据转到一个对象中

	@Test
	public void test1() throws SQLException{
		QueryRunner runner = new QueryRunner(JdbcUtils.getDataSource());
		String sql = "select * from users";
		Object result[] = (Object[]) runner.query(sql, new ArrayHandler());
		System.out.println(result[0]);
		System.out.println(result[1]);
	}
	
	@Test
	public void test2() throws SQLException{
		QueryRunner runner = new QueryRunner(JdbcUtils.getDataSource());
		String sql = "select * from users";
		List list = (List) runner.query(sql, new ArrayListHandler());
		System.out.println(list);
	}
	
	@Test
	public void test3() throws SQLException{
		QueryRunner runner = new QueryRunner(JdbcUtils.getDataSource());
		String sql = "select * from users";
		List list = (List) runner.query(sql, new ColumnListHandler1("name"));
		System.out.println(list);
	}
	
	
	@Test
	public void test4() throws SQLException{
		QueryRunner runner = new QueryRunner(JdbcUtils.getDataSource());
		String sql = "select * from users";
		Map<Integer,Map<String,Object>> map = (Map) runner.query(sql, new KeyedHandler("id"));

		for(Map.Entry<Integer,Map<String,Object>> me : map.entrySet()){
			int id = me.getKey();
			for(Map.Entry<String, Object> entry : me.getValue().entrySet()){
				String name = entry.getKey();
				Object value = entry.getValue();
				System.out.println(name + "=" + value);
			}
		}
	}
	
	@Test
	public void test5() throws SQLException{
		QueryRunner runner = new QueryRunner(JdbcUtils.getDataSource());
		String sql = "select count(*) from users";
		//Object result[] = (Object[]) runner.query(sql, new ArrayHandler());
		/*long totalrecord = (Long)result[0];
		int num = (int)totalrecord;
		System.out.println(num);
		int totalrecord = ((Long)result[0]).intValue();
		*/
		
		int totalrecord = ((Long)runner.query(sql, new ScalarHandler(1))).intValue();
		System.out.println(totalrecord);
	}

## dbUtils事务管理 ##
> 使用dbUtils处理事务，也很简单
> 
>- 在new QueryRunner对象时，不要传给它数据源，即，链接我们自己获得。
> 	QueryRunner runner = new QueryRunner();
>- 在使用QueryRunner.query() 或者 QueryRunner.update()。方法时使用带有Connection参数的重载形式，

	例如：	runner.update(JdbcUtils.getConnection(),sql, params);
			我们只需要不开启了事务的链接传递给它就OK了
	
	NT：
		我们自己获取的链接，不要忘记还给数据源，即调用close方法。
		事务相关的注意事项也不要忘记。