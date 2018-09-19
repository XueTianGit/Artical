title: Android-ContentProvide
date: 2016/2/29 8:24:32       
categories: Android
---

# Android-ContentProvide #

## 概述 ##
一般应用的数据库是不允许其他应用访问的，可是在某些情况下我们又是必须要访问的，比如一些联系人云备份应用。
Android为了解决这个问题，提供了ContentProvide。它作为android的四大组件之一。
内容提供者的作用：把私有数据暴露给其他应用，通常，是把私有数据库的数据暴露给其他应用。通过single ContentResolver interface.



## 自定义内容提供者 ##

1）自定义内容提供者，继承ContentProvider类，重写增删改查方法，在方法中写增删改查数据库的代码（这就是所谓的暴露）

例如增方法：

		@Override
		public Uri insert(Uri uri, ContentValues values) {
			db.insert("person", null, values);    //插入到person表
			return uri;
		}
2）在清单文件中定义内容提供者的标签

		<provider android:name="com.suixin.contentprovider.PersonProvider"
            android:authorities="com.suixin.person"
            android:exported="true"       
         ></provider>

> 注意： 必须要有authorities属性，这是内容提供者的主机名，功能类似地址。
>       android:exported="true" 必须写，否则暴露不成功

3）创建一个其他应用，访问自定义的内容提供者，实现对数据库的插入操作

例如：

		public void click(View v){
			//得到内容分解器对象
			ContentResolver cr = getContentResolver();    //想访问内容提供者的数据必须依赖 ContentResolver
			ContentValues cv = new ContentValues();
			cv.put("name", "小方");
			cv.put("phone", 138856);
			cv.put("money", 3000);
			//url:内容提供者的主机名
			cr.insert(Uri.parse("content://com.suixin.person"), cv);
		}

从上面代码可以看出，对内容提供者的访问时通过ContentResolver对象的，  即通过他的CRUD方法来访问内容提供者。

## 内容提供者的uri ##

android官方文档给出解释：
>- 每一个内容提供者通过一个uri（被URI对象封装）来唯一的确定其数据集合。
>- 当然，一个内容提供者会有许多不同数据（其实就是数据库有许多表），这体现在uri上就是对uri进行细分（/../..）。
>- 但是， 所有的内容提供者的uri都得以“content://”开头。  “content://” 就代表着数据是内容提供者的。

其实就是用来指明我们访问内容提供者的那些数据（因为内容提供在暴露很多数据的）。

看下面这个uri：

![](http://7xrbxa.com1.z0.glb.clouddn.com/android%E5%86%85%E5%AE%B9%E6%8F%90%E4%BE%9B%E8%80%85uri.png)

> - 内容提供者的uri开头必须写这个
> - 对应者清单文件中的authorities属性

 	<provider name=".TransportationProvider"          
 		authorities="com.example.transportationprovider"          . . .  >
> - 一般就是代表着表名（一个数据库可能会有多张表， 内容提供者依靠这一部分来区别不同的表呗）， 当然，我们在访问内容提供者时可以不写，但前提是该内容提供者提供者这样的uri。
> - 通常用作主键ID，当我们不访问特定一条数据时， 这个部分可以不写。 
 
	例如： “content://com.example.transportationprovider/trains/1”   一般都代表访问trains表的主键ID为1的数据。

### 给内容提供者添加URI ###
> 其实，如果我们不自己定义内容提供者的话，我们肯定不用去添加URI。但知道这个，可以方便我们去查看别人的内容提供者源代码，以便访问内容者的数据。

例如：

	UriMatcher.addURI("com.example.transportationprovider", "trains", 1);    
	我们在访问时，就可以写这个uri：content://com.example.transportationprovider/trains  ， 
	至于那个1， 是内容提供者自己分辨我们访问的是哪个uri（数据）。

例如一个内容提供者的代码：

	public class PersonProvider extends ContentProvider {
	
		private MyOpenHelper oh;
		SQLiteDatabase db;
	
		//创建uri匹配器对象
		static UriMatcher um = new UriMatcher(UriMatcher.NO_MATCH);
		//检测其他用户传入的uri与匹配器定义好的uri中，哪条匹配
		static {
			um.addURI("com.itheima.people", "person", 1);//content://com.itheima.people/person
			um.addURI("com.itheima.people", "teacher", 2);//content://com.itheima.people/teacher
			um.addURI("com.itheima.people", "person/#", 3);//content://com.itheima.people/person/4
		}
		
		//内容提供者创建时调用
		@Override
		public boolean onCreate() {
			oh = new MyOpenHelper(getContext());
			db = oh.getWritableDatabase();
			return false;
		}
	
		@Override
		public Cursor query(Uri uri, String[] projection, String selection,
				String[] selectionArgs, String sortOrder) {
			Cursor cursor = null;
			if(um.match(uri) == 1){
				cursor = db.query("person", projection, selection, selectionArgs, null, null, sortOrder, null);
			}
			else if(um.match(uri) == 2){
				cursor = db.query("teacher", projection, selection, selectionArgs, null, null, sortOrder, null);
			}
			else if(um.match(uri) == 3){
				//把uri末尾携带的数字取出来
				long id = ContentUris.parseId(uri);
				cursor = db.query("person", projection, "_id = ?", new String[]{id + ""}, null, null, sortOrder, null);
			}
			else{
				throw new IllegalArgumentException("uri又有问题哟亲么么哒");
			}
			return cursor;
		}
	
		@Override
		public String getType(Uri uri) {
			if(um.match(uri) == 1){
				return "vnd.android.cursor.dir/person";
			}
			else if(um.match(uri) == 3){
				return "vnd.android.cursor.item/person";
			}
			return null;
		}
	
		//此方法供其他应用调用，用于往people数据库里插数据
		//values：由其他应用传入，用于封装要插入的数据
		//uri:内容提供者的主机名，也就是地址
		@Override
		public Uri insert(Uri uri, ContentValues values) {
			//使用uri匹配器匹配传入的uri
			if(um.match(uri) == 1){
				db.insert("person", null, values);
				
				//发送数据改变的通知
				//uri:通知发送到哪一个uri上，所有注册在这个uri上的内容观察者都可以收到这个通知
				getContext().getContentResolver().notifyChange(uri, null);
			}
			else if(um.match(uri) == 2){
				db.insert("teacher", null, values);
				
				getContext().getContentResolver().notifyChange(uri, null);
			}
			else{
				throw new IllegalArgumentException("uri有问题哟亲么么哒");
			}
			return uri;
		}
	
		@Override
		public int delete(Uri uri, String selection, String[] selectionArgs) {
			int i = db.delete("person", selection, selectionArgs);
			return i;
		}
	
		@Override
		public int update(Uri uri, ContentValues values, String selection,
				String[] selectionArgs) {
			int i = db.update("person", values, selection, selectionArgs);
			return i;
		}
	
	}

## 内容观察者 ##
> 可以理解为一个监听器， 监听内容提供者的内容是否改变。

> 原理：
> 内容提供者在内容改变时，可以发出通知，通过ContentResolver.notifyChange(uri, null);
> uri:即把通知发送到这个uri上， 所有注册在这个uri上的内容观察者都可以收到这个通知

那么如何使用内容观察者呢？

1）注册内容观察者

	ContentResolver.registerContentObserver(Uri.parse("content://sms"), true, new MyObserver(new Handler()));

2)在MyObserver.onchange()方法中对内容提供者内容改变做相应的处理。







    
