title: Mybatis-逆向工程
date: 2016/3/9 15:47:14   
categories: WebFrame
---

# Mybatis-逆向工程 #

> Mybatis官方提供逆向工程，实现由数据库表生成mapper.xml、mapper.java、po类 及相关类。

想使用这个功能， 应首先去下载逆向工程相关的官方实现, 例如：
	mybatis-generator-core-1.3.2-buldle 

Mybatis的逆向工程，有好几种实现方式,例如使用eclipse插件，maven插件。
这里来看一下 java程序+XML配置，来完成mybatis逆向工程。

步骤：

>- 首先简单的创建一个java工程， 这个工程主要是为了存放我们生产的逆向工程文件，不要和我们的项目工程混到一块。


> - 添加逆向工程配置文件，在配置文件逆向工程配置信息，根据配置信息生成表相关的mapper文件，文件内容可以这样配置

	<!-- generatorConfiguration.xml --> 	
	<generatorConfiguration>
		<context id="testTables" targetRuntime="MyBatis3">
			<commentGenerator>
				<!-- 是否去除自动生成的注释 true：是 ： false:否 -->
				<property name="suppressAllComments" value="true" />
			</commentGenerator>
			<!--数据库连接的信息：驱动类、连接地址、用户名、密码 -->
			<jdbcConnection driverClass="com.mysql.jdbc.Driver"
				connectionURL="jdbc:mysql://localhost:3306/mybatis" userId="root"
				password="mysql">
			</jdbcConnection>
			<!-- <jdbcConnection driverClass="oracle.jdbc.OracleDriver"
				connectionURL="jdbc:oracle:thin:@127.0.0.1:1521:yycg" 
				userId="scott"
				password="tiger">
			</jdbcConnection> -->
	
			<!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL 和 
				NUMERIC 类型解析为java.math.BigDecimal -->
			<javaTypeResolver>
				<property name="forceBigDecimals" value="false" />
			</javaTypeResolver>
	
			<!-- targetProject:生成PO类的位置 -->
			<javaModelGenerator targetPackage="com.suixin.demo.po"
				targetProject=".\src">
				<!-- enableSubPackages:是否让schema作为包的后缀 -->
				<property name="enableSubPackages" value="false" />
				<!-- 从数据库返回的值被清理前后的空格 -->
				<property name="trimStrings" value="true" />
			</javaModelGenerator>
	        <!-- targetProject:mapper映射文件生成的位置 -->
			<sqlMapGenerator targetPackage="com.suixin.demo.dao.mapper" 
				targetProject=".\src">
				<!-- enableSubPackages:是否让schema作为包的后缀 -->
				<property name="enableSubPackages" value="false" />
			</sqlMapGenerator>
			<!-- targetPackage：mapper接口生成的位置 -->
			<javaClientGenerator type="XMLMAPPER"
				targetPackage="com.suixin.demo.dao.mapper" 
				targetProject=".\src">
				<!-- enableSubPackages:是否让schema作为包的后缀 -->
				<property name="enableSubPackages" value="false" />
			</javaClientGenerator>
			<!-- 指定数据库表 -->
			<table schema="" tableName="student"></table>
			
			<!-- 有些表的字段需要指定java类型
			 <table schema="" tableName="">
				<columnOverride column="" javaType="" />
			</table> -->
		</context>
	</generatorConfiguration>

可以看出， 在文件中，我们主要配置的是：

	数据库连接参数
	Po类存放位置
	Mapper映射文件和mapper接口的存放位置
	配置表名（其实，我们要的不就是这些东西吗）


> - 写java代码，执行逆向工程  （代码拷贝自官方文档）

	public class GeneratorSqlmap {
	
		public void generator() throws Exception{
	
			List<String> warnings = new ArrayList<String>();
			boolean overwrite = true;
			//指定 逆向工程配置文件
			File configFile = new File("generatorConfig.xml");   //知道这个就可以了。。。。。。。
			ConfigurationParser cp = new ConfigurationParser(warnings);
			Configuration config = cp.parseConfiguration(configFile);
			DefaultShellCallback callback = new DefaultShellCallback(overwrite);
			MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config,
					callback, warnings);
			myBatisGenerator.generate(null);
	
		} 
		public static void main(String[] args) throws Exception {
			try {
				GeneratorSqlmap generatorSqlmap = new GeneratorSqlmap();
				generatorSqlmap.generator();
			} catch (Exception e) {
				e.printStackTrace();
			}
			
		}
	
	}



> - 将生成的文件拷贝到我们，需要的地方



