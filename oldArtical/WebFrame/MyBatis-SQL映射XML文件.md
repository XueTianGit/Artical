title: MyBatis-SQL映射XML文件
date: 2016/3/9 9:19:20  
categories: WebFrame
---

# MyBatis-SQL映射XML文件 #

前面已经看过了这个文件中需要配一些什么东西，下面就SQL映射相关的元素来具体看一下。

## “#{}” ##
在看SQL映射之前，先看一下他是干什么的？
以这个配置为例：

	<!-- 例1-->
	<select id="findById" parameterType="int" resultType="com.suixin.mybatis.domain.Student">
		select id,name,sal from students where id = #{id}
	</select>
	<!-- 例2-->
	<insert id=”insertUser” parameterType=”User” >
		insert into users (id, username, password)
		values (#{id}, #{username}, #{password})
	</insert>

我们在代码中是这样写的：

	//例1
	Student student = sqlSession.selectOne(Student.class.getName()+".findById",id);   

> 在配置SQL映射时，对于参数的获取，是使用的它：#{}。
> parameterType=”User”，那么#{id}， 就会搜索User的id属性， 被使用他值

## select ##

> - 对于查询而言，sql语句书写先放一边， 我们要的当然是查询结果，上篇文章也已经看到了
> - MyBatis在操作数据库时，我们接收返回结果集很容易，而原因就是我们在这个配置文件中已经对结果
> 进行的映射。
> - 可以说MyBatis对查询结果集的封装，依赖于`<select/>`的 resultType 和resultMap决定。下面
> 看一下这两个属性是如何对查询结果集的封装的。

1）resultType：

	看这个配置：
		<select id="findById" parameterType="int" resultType="com.suixin.mybatis.domain.Student">
			select id,name,sal from students where id = #{id}
		</select>
	代码：
		Student student = sqlSession.selectOne(Student.class.getName()+".findById",id);   

MyBatis是如何将结果集封装到User中去的呢？

> 使用resultType，必须要求Student的属性必须和数据库表的字段一一对应。

对于集合查询：

	<select id="findAll" resultType="com.suixin.mybatis.domain.Student">
		select id,name,sal from students
	</select>
	List<Student> liststudents = sqlSession.selectList(Student.class.getName()+".findAll");

- resultMap

> 可以看出，resultType是有限制的。但是面对风云莫测的世界，resultMap允许我们对结果集的映射进行自己配置。

	<!--配置好结果映射集    property代表实体的属性， column代表数据库的字段-->
	<resultMap type="com.suixin.mybatis.domain.Student" id="studentMap">
		<id property="id" column="t_id"/>
		<result property="name" column="t_name"/>
		<result property="sal" column="t_sal"/>
	</resultMap>
	
	<！-- 引用我们自己配置的结果映射集-->
	<select id="findById" parameterType="int" resultMap="studentMap">
		select id,name,sal from students where id = #{id}
	</select>

	<select id="findAll" resultMap="studentMap">
		select id,name,sal from students
	</select>


## insert update delete ##

> 他们与`<select/>`有一点不同就是，没有提供resultType和resultType属性（也是合理的）
> 这三个标签在一些情况下， 即使和SQL语句的内容不符也是可以混用的。

其实也没有什么好讲，使用方法和`<select/>`类似， 看一下几个例子：

	<!-- 增加学生 -->
	<insert id="add" parameterType="com.suixin.mybatis.Student">
		insert into students(id,name,sal) values(#{id},#{name},#{sal})
	</insert>

	<!-- 更新学生 -->
	<update id="update" parameterType="com.suixin.mybatis.Student">
		update students set name=#{name},sal=#{sal} where id=#{id}
	</update>
	
	<!-- 删除学生 --> 
	<delete id="delete" parameterType="com.suixin.mybatis.Student">
		delete from students where id = #{id}
	</delete>

## sql片段 ##

<sql>这个元素用来定义能够被其它语句引用的可重用SQL语句块。
例如：

	<sql id="createcols">
		#{id},#{name},#{sal}
	</sql>

	<!-- 新增记录 -->
	<insert id="create" parameterType="com.suixin.mybatis.User">
		insert into students(id,name,sal) 
		values( <include refid="createcols"/> )
	</insert>








