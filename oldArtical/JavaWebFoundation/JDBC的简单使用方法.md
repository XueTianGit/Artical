title: JavaWebFoundation-JDBC（Java DataBase Connectivity）
date: 2016/3/1 9:18:40               
categories: JavaWebFoundation
---

# JDBC（Java DataBase Connectivity）  #
>- Sun公司为了简化对数据库的操作，定义了一套Java操作数据库的规范，称之为JDBC。
>- 既然是规范：
>   - Sun公司提供了JDBC的接口示范 ——JDBC API ，而数据库厂商或第三方中间厂商根据该
>     接口规范提供针对不同数据库的具体实现，称之为JDBC 驱动

- java开发时需要：
    - 组成JDBC的包： java.sql  javax.sql
    - JDBC驱动包：例如MySQL的包。

## JDBC操作数据库步骤简要 ##

1）加载驱动
	
	DriverManager.registerDriver(new com.mysql.jdbc.Driver());   

> 这样加载驱动并不好：
> 首先， 这样会导致加载了两次驱动
> 再者， 这样会导致程序依赖于mysql驱动的api， 为以后切换底层数据库带来麻烦
> 
> 一种比较好的做法：
> 	Class.forName(driverName);
> 	driverName我们可以从配置文件读取

2）获取链接

	Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/users", username, password);  

>- 数据库URL
>   - URL用于标识数据库的位置，程序员通过URL地址告诉JDBC程序连接哪个数据库
>   - URL的写法为：jdbc:mysql://localhost:3306/users？参数名：参数值
>   - 分别对应-----> 协对议  子协议    主机：端口    数据库
>   - 常用数据库URL地址的写法：
>      - Oracle写法：jdbc:oracle:thin:@localhost:1521:sid
>      - SqlServer—jdbc:microsoft:sqlserver://localhost:1433; DatabaseName=sid
>      - MySql—jdbc:mysql://localhost:3306/sid  -> 
>        Mysql的url地址的简写形式： jdbc:mysql:///sid


3） 获取向数据库发送sql语句的statement对象
	
	Statement st = conn.createStatement();
	
4） 向数据库发送sql语句，并获取返回的结果集

	ResultSet rs = st.executeQuery("select * from users");

5） 从结果集中获取数据
	
	while(rs.next()){
		Sop(rs.getObject("id"));
	}
	
6）释放链接

	释放链接的代码，一般写在finally中，因为必须释放！！！
		finally
		{
			//release resource
			if( rts != null ){
				try
				{
					rts.close();
				}
				catch( SQLException e)
				{
					e.printStackTrace();
				}
			}
			
			if( st != null ){
				try
				{
					st.close();
				}
				catch( SQLException e)
				{
					e.printStackTrace();
				}
			}

			if( con != null ){
				try
				{
					con.close();	
				}
				catch( SQLException e)
				{
					e.printStackTrace();
				}
			}
		
		}

## JDBC中的对象 ##

* Connection
 
> Jdbc程序中的Connection，它用于代表数据库的链接，Collection是数据库编程中
> 最重要的一个对象，客户端与数据库所有交互都是通过connection对象完成的，
> 这个对象的常用方法：

	  createStatement()：创建向数据库发送sql的statement对象。
	  prepareStatement(sql) ：创建向数据库发送预编译sql的PrepareSatement对象。
	  prepareCall(sql)：创建执行存储过程的callableStatement对象。
	  setAutoCommit(boolean autoCommit)：设置事务是否自动提交。
	  commit() ：在链接上提交事务。
	  rollback() ：在此链接上回滚事务。
 
* Statement
 
> Jdbc程序中的Statement对象用于向数据库发送SQL语句， Statement对象常用方法：
 
	   executeQuery(String sql) ：用于向数据发送查询语句。
	   executeUpdate(String sql)：用于向数据库发送insert、update或delete语句
	   execute(String sql)：用于向数据库发送任意sql语句
	   addBatch(String sql) ：把多条sql语句放到一个批处理中。
	   executeBatch()：向数据库发送一批sql语句执行。
 
* ResultSet
 
> Jdbc程序中的ResultSet用于代表Sql语句的执行结果。Resultset封装执行结果时，采用的类似于表格的方式。ResultSet 对象维护了一个指向表格数据行的游标，初始的时候，游标在第一行之前，调用ResultSet.next() 方法，可以使游标指向具体的数据行，进行调用方法获取该行的数据。
> ResultSet既然用于封装执行结果的，所以该对象提供的都是用于获取数据的get方法：
	 
	获取任意类型的数据
	    getObject(int index)
	    getObject(string columnName)
	获取指定类型的数据，例如：
	    getString(int index)
	    getString(String columnName)
	 
	ResultSet还提供了对结果集进行滚动的方法：
	   next()：移动到下一行
	   Previous()：移动到前一行
	   absolute(int row)：移动到指定行
	   beforeFirst()：移动resultSet的最前面。
	   afterLast() ：移动到resultSet的最后面。

* PrepareStatement

> 与Statement的区别： Statement执行一条sql就得编译一次，PrepareStatement只编译一次；常用后者原因在于参数设置非常方便；对sql语句的预编译，会减轻数据库的压力

- 为何使用它：
   - 代码的可读性和可维护性.

> 虽然用PreparedStatement来代替Statement会使代码多出几行,但这样的代码无论从可读性还是可维护性上来说.都比直接用Statement的代码高很多档次:

	example：
	ps = con.prepareStatement("insert into tb_name (col1,col2,col2,col4) values (?,?,?,?)");
	ps.setString(1,var1);
	ps.setString(2,var2);
	ps.setString(3,var3);
	ps.setString(4,var4);
	ps.executeUpdate();

   - PreparedStatement尽最大可能提高性能.
  
> 每一种数据库都会尽最大努力对预编译语句提供最大的性能优化.**因为预编译语句有可能被重复调用.所以语句在被DB的编译器编译后的执行代码被缓存下来,那么 下次调用时只要是相同的预编译语句就不需要编译,只要将参数直接传入编译过的语句执行代码中(相当于一个涵数)就会得到执行**.这并不是说只有一个 Connection中多次执行的预编译语句被缓存,而是对于整个DB中,只要预编译的语句语法和缓存中匹配.那么在任何时候就可以不需要再次编译而可以 直接执行.而statement的语句中,即使是相同一操作,**而由于每次操作的数据不同所以使整个语句相匹配的机会极小,几乎不太可能匹配**.

	比如:
	  insert into tb_name (col1,col2) values ('11','22');
	  insert into tb_name (col1,col2) values ('11','23');

> 即使是相同操作但因为数据内容不一样,所以整个个语句本身不能匹配,没有缓存语句的意义.事实是没有数据库会对普通语句编译后的执行代码缓存.
>  当然并不是所以预编译语句都一定会被缓存,数据库本身会用一种策略,比如使用频度等因素来决定什么时候不再缓存已有的预编译结果.以保存有更多的空间存储新的预编译语句.

- 防止sql注入攻击

> 即使到目前为止,仍有一些人连基本的恶义SQL语法都不知道.

	String sql = "select * from tb_name where name= '"+varname+"' and passwd='"+varpasswd+"'";
	如果我们把[' or '1' = '1]作为varpasswd传入进来.用户名随意,看看会成为什么?

	select * from tb_name = '随意' and passwd = '' or '1' = '1';
	因为'1'='1'肯定成立,所以可以任何通过验证.更有甚者:
	把[';drop table tb_name;]作为varpasswd传入进来,则:
	select * from tb_name = '随意' and passwd = '';drop table tb_name;有些数据库是不会让你成功的,但也有很多数据库就可以使这些语句得到执行.

> 而如果你使用预编译语句.你传入的任何内容就不会和原来的语句发生任何匹配的关系.只要全使用预编译语句,你就用不着对传入的数据做任何过虑.而如果使用普通的statement,有可能要对drop,;等做费尽心机的判断和过虑.