title: Mybatis-延迟加载和缓存机制
date: 2016/3/9 15:46:58   
categories: WebFrame
---

# Mybatis-延迟加载和缓存机制 #

## 延迟加载 ##
> 这里说的延迟加载与多表查询有关，在需求允许的情况下，先查询单表，当需要关联其它表查询时，进行延迟加载，去关联查询。
> 即:不需要关联信息时不查询，需要时再查询。
> 好处当然是能够提升数据库的性能。

使用步骤：

> 在mybatis.xml中配置<setting>全局参数

	lazyLoadingEnabled：延迟加载的总开关，设置为true
	aggressiveLazyLoading：设置为false，实现按需加载（将积极变为消极）
	
	例如：
	<settings>
		<setting name="lazyLoadingEnabled" value="true">
		<setting name="aggressiveLazyLoading" value="false">
	</settings>

> 在SQL映射文件中配置resultMap的延迟加载


	resultMap是对订单的映射，当我们查询订单时，不同时将User查询出来。

	<!-- 订单及用户的resultMap，实现延迟加载 -->
	<resultMap type="orders" id="ordersResultMap">
		<!-- 配置订单信息的映射 -->
		<id column="id" property="id" />
		<result column="user_id" property="user_id" />
		<result column="order_number" property="order_number" />

		<!-- 配置延迟加载 用户信息
			 select：延迟加载 时调用 的statement(SQL)，如果跨命名空间，需要加上namespace 
			 column：将哪一列的值作为参数 传到延迟加载 的SQL, 即延迟加载时靠的是什么条件 -->
		<association property="user" javaType="com.suixin.mybatis.po.User"
			select="com.suixin.mybatis.domain.user.findUserById" column="user_id">
		</association>
	</resultMap>


## 缓存机制 ##
使用缓存的目的：
将从数据库中查询出来的数据缓存起来（缓存介质：内存、磁盘），当再次使用数据时，从缓存中取数据，而不从数据库查询，
减少了数据库的操作，提高了数据处理性能。

**如果二缓存开启，首先从二级缓存查询数据，如果二级缓存有则从二级缓存中获取数据，如果二级缓存没有，从一级缓存找是否有缓存数据，如果一级缓存没有，查询数据库。**

### 一级缓存 ###

> Mybatis默认提供一级缓存，缓存范围是一个sqlSession。（类似Hibernate）
> 在同一个SqlSession中，两次执行相同的sql查询，第二次不再从数据库查询。

注意事项：
**如果第一次查询后，执行commit提交，mybatis会清除缓存，第二次查询还是会去数据库查询。**

### 二级缓存 ###

> 缓存范围是跨SqlSession的，范围是mapper的namespace，相同的namespace使用一个二级缓存结构。需要进行参数配置让mybatis支持二级缓存。

配置步骤：

	1）在mybatis的中配置文件中打开二级缓存开关。
		<setting name="cacheEnabled" value="true"/>
	2）在SQL映射文件中，打开该mapper的二级缓存。
		<mapper namespace="xxx">
			<cache/>   <!-- 打开给mapper的二级缓存-->
		</mapper>
	3）针对某个SQL映射的二级缓存配置
	a，让某个statement启用二级缓存（默认是开启的）
		<select  。。。。 useCache="true"> </select>
	b， 让statement执行后刷新缓存（清除缓存）默认是开启的）
		<select  。。。。 flushCache="true"> </select>	

#### 二级缓存应用场景 ####

1、针对复杂的查询或统计的功能，用户不要求每次都查询到最新信息，使用二级缓存，通过刷新间隔flushInterval设置刷新间隔时间，由mybatis自动刷新。
比如：实现用户分类统计sql，该查询非常耗费时间。
将用户分类统计sql查询结果使用二级缓存，同时设置刷新间隔时间：flushInterval（一般设置时间较长，比如30分钟，60分钟，24小时，根据需求而定）

->flushInterval为<cache/>的属性


2、针对信息变化频率高，需要显示最新的信息，使用二级缓存。
将信息查询的statement与信息的增、删、改定义在一个mapper.xml中，此mapper实现二级缓存，当执行增、删、修改时，由mybatis及时刷新缓存，满足用户从缓存查询到最新的数据。
比如：新闻列表显示前10条，该查询非常快，但并发大对数据也有压力。
将新闻列表查询前10条的sql进行二级缓存，这里不用刷新间隔时间，当执行新闻添加、
 不过这个需求的最佳方案是使用页面缓存



