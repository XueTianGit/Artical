title: MyBatis-动态SQL
date: 2016/3/9 9:19:20  
categories: WebFrame
---

# MyBatis-动态SQL #

- 何为动态SQL呢？
简单的说就是： 查询条件不确定，需要根据情况产生SQL语句，这种情况叫动态SQL。

- 为什么需要动态SQL呢：
你应该明白把SQL语句条件连接在一起是多么的痛苦，要确保不能忘记空格或者不要在
columns列后面省略一个逗号等。动态语句能够完全解决掉这些痛苦。

- 既然是动态的产生SQL语句， 那就代表着我们在SQL映射文件中的SQL语句应该写活的，
那么怎么写活呢？

	MyBatis提供了以下几个标签来帮助我们实现动态SQL：
	`<if> <trim> <where> <set> <choose> <when> <otherwise>  <foeach>`

来看一下他们怎么使用

## if ##
> 动态 SQL 最常做的事就是有条件地包括 where 子句.
> 在下面这个例子中，我们可以动态的更改where条件， **对于SQL语句的拼接， 则由MyBatis完成.**

	select * from students from where后面的条件不确定。
	
	<!--动态SQL解决方案-->
	<select id="findAll" parameterType="map" resultMap="studentMap">
		select * from students
		<where>
			<if test="pid!=null">    <!-- pid对应着map中的key-->
				and students_id = #{pid}
			</if>
			<if test="pname!=null">
				and students_name = #{pname}
			</if>
			<if test="psal!=null">
				and students_sal = #{psal}
			</if>
		</where>
	</select>

	代码：
			Map<String,Object> map = new LinkedHashMap<String,Object>();
			map.put("pid",id);
			map.put("pname",name);
			map.put("psal",sal);	
			sqlSession.selectList("studentNamespace.findAll",map);  //studentNamespace， 我们把名称空间写死了

## set ##
> 问题：在更新数据库时字段不确定


	update students 
	<!-- set标签自动判断哪个是最后一个字段，会自动去掉最后一个,号 -->
	<update id="dynaUpdate" parameterType="map">
		update students 
		<set>
			<if test="pname!=null">
				students_name = #{pname},
			</if>
			<if test="psal!=null">
				students_sal = #{psal},			
			</if>
		</set>
		where students_id = #{pid}
	</update>

## foreach ##
> 问题：在删除数据时 in(...)中的值不确定

	<delete id="dynaDeleteArray">
		delete from students where students_id in
		<!-- foreach用于迭代数组元素
			 open表示开始符号
			 close表示结束符合
			 separator表示元素间的分隔符
			 item表示迭代的数组，属性值可以任意，但提倡与方法的数组名相同
			 #{ids}表示数组中的每个元素值
		 -->
		<foreach collection="array" open="(" close=")" separator="," item="ids">
			#{ids}
		</foreach>
	</delete>
	代码：
	sqlSession.delete("studentNamespace.dynaDeleteArray",ids);

> 可以看出<foreach>不仅能解决in(..)的问题

## trim ##
> 我们主要使用这个元素来解决：当MyBatis不能帮我们很好的拼接SQL语句时。
例如，在SQL片段中：

	<sql id="key">
		<!-- 去掉最后一个, -->
		<trim suffixOverrides=",">
			<if test="id!=null">
				students_id,
			</if>
			<if test="name!=null">
				students_name,
			</if>
			<if test="sal!=null">
				students_sal,
			</if>
		</trim>
	</sql>

我们还可以使用太来完成where的功能：
下面两个SQL映射， 结果是一样的：

	<select id=”findActiveBlogLike” parameterType=”Blog” resultType=”Blog”>
		SELECT * FROM BLOG
		<where>
			<if test=”state != null”>
				state = #{state}
			</if>
			<if test=”title != null”>
				AND title like #{title}
			</if>
			<if test=”author != null and author.name != null”>
				AND title like #{author.name}
			</if>
		</where>
	</select>
	
	<select id=”findActiveBlogLike” parameterType=”Blog” resultType=”Blog”>
		SELECT * FROM BLOG
		< trim prefix= "WHERE" prefixOverrides= "AND |OR " > 
			<if test=”state != null”>
				state = #{state}
			</if>
			<if test=”title != null”>
				AND title like #{title}
			</if>
			<if test=”author != null and author.name != null”>
				AND title like #{author.name}
			</if>
		</trim > 
	</select>

