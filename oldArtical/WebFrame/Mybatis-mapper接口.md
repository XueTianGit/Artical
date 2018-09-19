title: Mybatis-mapper接口
date: 2016/3/9 9:19:20  
categories: WebFrame
---


# Mybatis-mapper接口#

回顾一下，前面我们代码的编写，我们根据SQL映射文件中的XML定义：

	SqlSession sqlSession = sqlSessionFactory.openSession();		
	Map<String,Object> map = new LinkedHashMap<String,Object>();
	map.put("pid",id);
	map.put("pname",name);
	map.put("psal",sal);
	sqlSession.selectList("studentNamespace.findAll",map);  //名称空间.id 来引用我们在XML文件中定义的SQL语句


下面来看一下mapper接口（mapper动态代理）的方式书写上面这段代码：

	SqlSession sqlSession = sqlSessionFactory.openSession();
	StudentMapperCustom ordersMapperCustom = sqlSession.getMapper(StudentMapperCustom.class);  //使用mapper接口的方式来引用我们定义的XML语句
	Map<String,Object> map = new LinkedHashMap<String,Object>();
	map.put("pid",id);
	map.put("pname",name);
	map.put("psal",sal);
	List<OrdersCustom> list = ordersMapperCustom.findAll(map);


> 那么使用mapper接口的方式来引用我们定义的XML语句怎么实现呢？



> 先来看以下SQL映射XML文件中的定义

	<!--UserMapper.xml-->
	<mapper namespace="com.suixin.mybatis.mapper.UserMapper">
	
		<select id="findUserById" parameterType="int" resultType="user" >
			SELECT * FROM USER WHERE id = #{id}
		</select>
	
		<select id="findUserList" parameterType="java.lang.String" resultType="com.suixin.mybatis.po.User" >
		  SELECT * FROM USER WHERE username LIKE '%${value}%'
		</select>
		
		<update id="updateUser" parameterType="com.suixin.mybatis.po.User" >
		   update user set username=#{username},birthday=#{birthday},sex=#{sex}  where id=#{id}
		</update>
		
	</mapper>

> 看一下， 我们依据SQL映射文件， 提供出来的接口：

	<!--UserMapper.java-->
	public interface UserMapper {
		
		public User findUserById(int id) throws Exception;
		
		public List<User> findUserList(String username)throws Exception;
		
		public void updateUser(User user) throws Exception;
	
	}

> 代码测试是这样写的

 	@Test
	public void testFindUserById() throws Exception {
		SqlSession sqlSession = sqlSessionFactory.openSession();
		UserMapper userMapper = sqlSession.getMapper(UserMapper.class);  
		User user= userMapper.findUserById(10);
		System.out.println(user);
	}


- 综上， 我们可以看出，这种方法， 就是把我们SQL映射文件中的 SQL语句（操作），包装成了Java方法， 以便我们调用。
- 其实这和使用XML：	sqlSession.selectList("studentNamespace.findAll",map);  这种方式，思想肯定是一样的：
- 即：** 我们要在代码中执行我们在XML文件中的SQL语句**，  只不过mapper接口使用起来更方便些而已。
> 

	mapper接口使用需要注意的点：
	1. 接口文件的名称必须要和SQL映射XML文件相同， 这样MyBatis才能找到我们定义的SQL语句
	2. 	UserMapper userMapper = sqlSession.getMapper(UserMapper.class);       这个返回的UserMapper实际上是一个代理。(使用的是JDK代理)
	3. <mapper namespace="com.suixin.mybatis.mapper.UserMapper">   namespace 就是map接口的地址


## mapper接口加注解 ##
> 如果嫌XML文件太麻烦， 还可以使用mapper接口加注解， （我只是了解一下，）


>定义接口文件

	package com.suixin.mybatis.mapperinterface. 
	public interface UserMapper {  
	   
	    @Insert("insert into t_user(name, age) values(#{name}, #{age})")  
	    public void insertUser(User user);  
	     
	    @Update("update t_user set name=#{name}, age=#{age} where id=#{id}")  
	    public void updateUser(User user);  
	     
	    @Select("select * from t_user where id=#{id}")  
	    public User findById(int id);  
	     
	    @Delete("delete from t_user where id=#{id}")  
	    public void deleteUser(int id);  
	     
	}  

>Mapper信息注册到Mybatis的配置中，告诉Mybatis我们定义了哪些Mapper信息。

	<mappers>  
	   <mapper class="com.suixin.mybatis.mapperinterface.UserMapper"/>  
	</mappers>  

> 如果感觉一个一个接口的指定太麻烦，还可以一次指定一个包：  （这样Mybatis就会把这个包下面的所有Mapper接口都进行注册）

	<mappers>  
	   <package name="com.suixin.mybatis.mapperinterface"/>  
	</mappers>  

> 在代码中的使用方法和XML+mapper接口一样

**以上只是我对mybatis的简单了解**