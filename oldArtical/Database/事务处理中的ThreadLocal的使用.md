title: 事务处理中的ThreadLocal的使用
date: 2016/4/22 8:45:55  
categories: Database
---

# 事务处理中的ThreadLocal的使用 #

## 简单的理解ThreadLocal ##
> 所谓ThreadLocal，简单一点想，就是一个全局的Map，Map的key是线程对象，value是你要保存的对象
> 进入某个线程后，就可以从map中取得之前存储的相应线程关联的对象。

> NT：ThreadLocal并不是一个Map，但用Map来理解是没有问题的

## 在事务中使用ThreadLocal ##

> 确保所有的sql语句都在同一个开启了事务的链接上执行，这时候就可以使用ThreadLocal来解决这个问题。

代码实现：

	//绑定开启了链接的connection到ThreadLocal的JdbcUtils工具类
	public class JdbcUtils {
	
		private static ThreadLocal<Connection> tl = new ThreadLocal<Connection>();  //map
	
	
		public static void startTransaction(){
			try{
				//得到当前线程上绑定连接开启事务
				Connection conn = tl.get();
				if(conn==null){  //代表线程上没有绑定连接
					conn = ds.getConnection();    //ds 代表数据源， 这里省略他的代码
					tl.set(conn);
				}
				conn.setAutoCommit(false);
			}catch (Exception e) {
				throw new RuntimeException(e);
			}
		}
		
		
		public static void commitTransaction(){
			try{
				Connection conn = tl.get();
				if(conn!=null){
					conn.commit();
				}
			}catch (Exception e) {
				throw new RuntimeException(e);
			}
		}
		
	}
	
	
	
	//在服务层用上ThreadLocal的事务管理
	public void transfer2(int sourceid,int targetid,double money) throws SQLException{
		try{
			JdbcUtils.startTransaction();
			AccountDao dao = new AccountDao();
			Account a = dao.find(sourceid);   //select
			Account b = dao.find(targetid);   //select
			a.setMoney(a.getMoney()-money);  
			b.setMoney(b.getMoney()+money);   
			dao.update(a); //update
			dao.update(b);//update
			JdbcUtils.commitTransaction();
		}finally{
			JdbcUtils.closeConnection();
		}
	}
	}
	
	//持久层，获取绑定在当前线程上的connection，进行操作
	
	public Account find(int id){
			try{
				QueryRunner runner = new QueryRunner();
				String sql = "select * from account where id=?";
				return (Account) runner.query(JdbcUtils.getConnection(),sql, id, new BeanHandler(Account.class));
			}catch (Exception e) {
				throw new RuntimeException(e);
			}
		}


## 使用ThreadLocal应该注意的问题 ##

在上面的代码中将ThreadLocal对象做成了静态

    private static ThreadLocal<Connection> tl = new ThreadLocal<Connection>();  //map

因此，每次开启一条线程，ThreadLocal都会有一个Connection加进来，所以，我们要记得将这些Connection移除，不然
这个ThreadLocal静态对象会越变越大，最后导致程序挂掉！！！！！！

-> 即，在关闭链接是，把变量移除

	public static void closeConnection(){
		try{
			Connection conn = tl.get();
			if(conn!=null){
				conn.close();
			}
		}catch (Exception e) {
			throw new RuntimeException(e);
		}finally{
			tl.remove();   //千万注意，解除当前线程上绑定的链接（从threadlocal容器中移除对应当前线程的链接）
		}
	}
