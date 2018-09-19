title: Android中的SQLite数据库的简单使用
date: 2016/2/28 12:20:20  
categories: Android
---

# Android中的SQLite数据库的简单使用 #

## 什么是SQLite ##

> - SQLite，是一款轻型的数据库，是遵守ACID(原子性、一致性、隔离性、持久性)的关联式数据库管理系统，多用于嵌入式开发中。
> - 它是D.Richard Hipp用C语言编写的开源嵌入式数据库引擎。它支持大多数的SQL92标准，并且可以在所有主要的操作系统上运行。
> - SQLite由以下几个部分组成：SQL编译器、内核、后端以及附件。SQLite通过利用虚拟机和虚拟数据库引擎(VDBE)，使调试、修改
> 和扩展SQLite的内核变得更加方便。所有SQL语句都被编译成易读的、可以在SQLite虚拟机中执行的程序集。
> - 现在的主流移动设备像Android、iPhone等都使用SQLite作为复杂数据的存储引擎。


##  SQLite的数据类型 ##
SQLite的数据类型：Typelessness(无类型), 可以保存任何类型的数据到你所想要保存的任何表的任何列中. 

>- SQLite采用动态数据类型，当某个值插入到数据库时，SQLite将会检查它的类型，如果该类型与关联的列不匹配，
>- SQLite则会尝试将该值转换成该列的类型，如果不能转换，则该值将作为本身的类型存储，SQLite称这为“弱类型”。
>- 但有一个特例，如果是INTEGER PRIMARY KEY，则其他类型不会被转换，会报一个“datatype missmatch”的错误。
>- 概括来讲，SQLite支持NULL、INTEGER、VARCHAR、REAL、TEXT和BLOB数据类型，分别代表空值、整型值、浮点值、字符串文本、二进制对象。


## 使用 ##

### 创建数据库 ###

1) 创建SQLiteOpenHelper抽象类的实现类

相关API：SQLiteOpenHelper抽象类，该类用于对数据库版本进行管理.该类中常用的方法:

	onCreate	数据库创建时执行(第一次连接获取数据库对象时执行)
	onUpgrade	数据库更新时执行(版本号改变时执行)
	onOpen		数据库每次打开时执行(每次打开数据库时调用，在	onCreate，onUpgrade方法之后)

由于它是一个抽象类，所以我们必须自定义他的实现类。：

	public class MyOpenHelper extends SQLiteOpenHelper {

		//name:数据库文件的名字
		//factory:游标工厂
		//versio:数据库版本
		//
		public MyOpenHelper(Context context, String name, CursorFactory factory,
				int version) {
			super(context, name, factory, version);
			// TODO Auto-generated constructor stub
		}
	
		//数据库创建时，此方法会调用
		@Override
		public void onCreate(SQLiteDatabase db) {
			db.execSQL("create table person(_id integer primary key autoincrement, name char(10), salary char(20), phone integer(20))");
	
		}
	
		//数据库升级时，此方法会调用
		@Override
		public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
			System.out.println("数据库升级了");
		}
		
	}

> NT: 上面这个类的3个方法， 我们一般都要定义。

2）创建数据库

		//getContext():获取一个虚拟的上下文，  
		MyOpenHelper oh = new MyOpenHelper(this, "people.db", null, 1);
		//如果数据库不存在，先创建数据库，再获取可读可写的数据库对象，如果数据库存在，就直接打开
		SQLiteDatabase db = oh.getWritableDatabase();


>- SQLiteOpenHelper.getWritableDatabase()：打开可读写的数据库
>- SQLiteOpenHelper.getReadableDatabase()：在磁盘空间不足时打开只读数据库，否则打开可读写数据库

### 创建表 ###
> 如上面代码，我们在数据库创建时，创建了表：

	db.execSQL("create table person(_id integer primary key autoincrement, name varchar(10), salary varchar(20), phone integer(20))");

### 执行CRUD操作 ###
#### SQL语句实现 ####

	public void insert(){	
		db.execSQL("insert into person (name, salary, phone) values(?, ?, ?)", new Object[]{"霸天虎", "13000", 138438});
		db.execSQL("insert into person (name, salary, phone) values(?, ?, ?)", new Object[]{"威震天", 14000, "13888"});
		db.execSQL("insert into person (name, salary, phone) values(?, ?, ?)", new Object[]{"吊炸天", 14000, "13888"});
	}
	
	public void delete(){
		db.execSQL("delete from person where name = ?", new Object[]{"霸天虎"});
	}
	
	public void update(){
		db.execSQL("update person set phone = ? where name = ?", new Object[]{186666, "吊炸天"});
	}
	
	public void select(){
		Cursor cursor = db.rawQuery("select name, salary from person", null);

		while(cursor.moveToNext()){
			//通过列索引获取列的值
			String name = cursor.getString(cursor.getColumnIndex("name"));
			String salary = cursor.getString(1);
			System.out.println(name + ";" + salary);
		}
	}


> 很明显，只是API不同，大致操作和其他数据库的步骤相似。
> Cursor类似于MySQL数据库的ResultSet，是不是！

#### API实现 ####
* 插入

		//以键值对的形式保存要存入数据库的数据
		ContentValues cv = new ContentValues();
		cv.put("name", "刘能");
		cv.put("phone", 1651646);
		cv.put("money", 3500);
		//返回值是改行的主键，如果出错返回-1
		long i = db.insert("person", null, cv);   //插入到person表
* 删除

		//返回值是删除的行数
		int i = db.delete("person", "_id = ? and name = ?", new String[]{"1", "张三"});
* 修改

		ContentValues cv = new ContentValues();
		cv.put("money", 25000);
		int i = db.update("person", cv, "name = ?", new String[]{"赵四"});
* 查询

		//arg1:要查询的字段
		//arg2：查询条件
		//arg3:填充查询条件的占位符
		Cursor cs = db.query("person", new String[]{"name", "money"}, "name = ?", new String[]{"张三"}, null, null, null);
		while(cs.moveToNext()){
			//							获取指定列的索引值
			String name = cs.getString(cs.getColumnIndex("name"));
			String money = cs.getString(cs.getColumnIndex("money"));
			System.out.println(name + ";" + money);
		}

###事务
-> 事务api

		try {
			//开启事务
			db.beginTransaction();
			...........
			//设置事务执行成功
			db.setTransactionSuccessful();
		} finally{
			//关闭事务
			//如果此时已经设置事务执行成功，则sql语句生效，否则不生效
			db.endTransaction();
		}

