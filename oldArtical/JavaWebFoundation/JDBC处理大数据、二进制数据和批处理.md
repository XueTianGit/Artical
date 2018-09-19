title: JavaWebFoundation-JDBC处理大数据、二进制数据和批处理
date: 2016/3/1 9:18:40               
categories: JavaWebFoundation
---

# JDBC处理大数据、二进制数据和批处理 #


## 使用JDBC处理大数据 ##
例如，文本数据
> 存储

	PrepareStatement.setCharacterStream(index, fis, length);
    index： 是预编译sql语句中大文本数据的参数索引位置
	fis：大文本数据的输入流
	length：大文本数据的长度

> 获取：

	方法一：
		reader = resultSet.getCharacterStream("article");   //article为列名啦
	方法二：
		reader = resultSet.getBlob(i).getCharacterStream("article");  //以Bolb对象获取当前结果集的指定列，然后获取流
	方法三：
		String data = resultSet.getString("article");  //吊炸天
	
	范例-> 处理ASCLL数据
	     //Open a FileInputStream
	      File f = new File("XML_Data.xml");
	      long fileLength = f.length();
	      FileInputStream fis = new FileInputStream(f);
	
	      //Create PreparedStatement and stream data
	      String SQL = "INSERT INTO XML_Data VALUES (?,?)";
	      pstmt = conn.prepareStatement(SQL);
	      pstmt.setInt(1,100);
	      pstmt.setAsciiStream(2,fis,(int)fileLength);
	      pstmt.execute();
	
	      //Close input stream
	      fis.close();
	
	      // Do a query to get the row
	      SQL = "SELECT Data FROM XML_Data WHERE id=100";
	      rs = stmt.executeQuery (SQL);
	      // Get the first row
	      if (rs.next ()){
	         //Retrieve data from input stream
	         InputStream xmlInputStream = rs.getAsciiStream (1);
	         int c;
	         ByteArrayOutputStream bos = new ByteArrayOutputStream();
	         while (( c = xmlInputStream.read ()) != -1)
	            bos.write(c);
	         //Print results
	         System.out.println(bos.toString());
	      }

## 使用JDBC处理二进制数据 ##
> 存储：

	PrepareStatement.setBinaryStream(index, fis, length);
> 获取：

	resultSet.getBinaryStream("binarydata");

## 使用JDBC进行批处理 ##

> 业务场景：当需要向数据库发送一批SQL语句执行时，应避免向数据库一条条的发送执行
> 而应采用JDBC的批处理机制，以提升执行效率。

实现批处理有两种方式，第一种方式：
	
	Statement.addBatch(sql)  list
	
	执行批处理SQL语句
	
	executeBatch()方法：执行批处理命令
	
	clearBatch()方法：清除批处理命令
	
	example:
	
	Connection conn = null;
	
	Statement st = null;
	
	ResultSet rs = null;
	
	try {
	
		conn = JdbcUtil.getConnection();
		
		String sql1 = "insert intouser(name,password,email,birthday)
		
		      values('kkk','123','abc@sina.com','1978-08-08')";
		
		String sql2 = "update user setpassword='123456' where id=3";
		
		st = conn.createStatement();
		
		st.addBatch(sql1);  //把SQL语句加入到批命令中
		
		st.addBatch(sql2);  //把SQL语句加入到批命令中
		
		st.executeBatch();
	
	} finally{
	
	      JdbcUtil.free(conn,st, rs);
	
	}

> 采用Statement.addBatch(sql)方式实现批处理时：

- 优点：可以向数据库发送多条不同的ＳＱＬ语句。
- 缺点：
  - SQL语句没有预编译。
  - 当向数据库发送多条语句相同，但仅参数不同的SQL语句时，需重复写上很多条SQL语句。

 

使用批量处理的第二种方式

	conn = JdbcUtil.getConnection();
	
	String sql = "insert intouser(name,password,email,birthday) values(?,?,?,?)";
	
	st = conn.prepareStatement(sql);
	
		for(int i=0;i<50000;i++){
		
			st.setString(1,"aaa" + i);
			
			st.setString(2,"123" + i);
			
			st.setString(3,"aaa" + i + "@sina.com");
			
			st.setDate(4,new Date(1980, 10,10));

			st.addBatch();
			
			if(i%1000==0){
			
				st.executeBatch();				
				st.clearBatch();
		
		}
	
	}
	
	st.executeBatch();

 

> 采用PreparedStatement.addBatch()实现批处理

- 优点：发送的是预编译后的SQL语句，执行效率高。
- 缺点：只能应用在SQL语句相同，但参数不同的批处理中。因此此种形式的批处理经常用于在同一个表中批量插入数据，或批量更新表的数据。

 