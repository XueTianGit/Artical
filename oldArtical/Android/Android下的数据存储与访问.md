title: Android下的数据存储与访问
date: 2016/2/28 12:20:20  
categories: Android
---

# Android下的数据存储与访问 #

文件的存储共有5种方式:

* 文件
* SharedPrefrence
* SQLite数据库
* Content provider
* 网络

## 文件 ##

### 使用文件进行数据存储 ###
> 在上下文中有一个方法叫openFileOutput()方法可以用于把数据输出到文件中，具体的实现过程与在J2SE环境中保存数据到文件中是一样的。

	FileOutputStream outStream = this.openFileOutput("Susion.txt", Context.MODE_PRIVATE);
	outStream.write("巧克力键盘".getBytes());
	outStream.close();   
	openFileOutput()方法的第一参数用于指定文件名称，不能包含路径分隔符“/” ，如果文件不存在，Android会自动创建它。
	
	创建的文件保存在/data/data/<package name>/files目录，如：/data/data/cn.itcast/files/Susion.txt ，通过点击Eclipse菜单“Window”-“Show View”-“Other”，在对话窗口中展开android文件夹，选择下面的File Explorer视图，然后在File Explorer视图中展开/data/data/<package name>/files目录就可以看到该文件。

- openFileOutput()方法的第二参数用于指定操作模式，有四种模式，分别为： 

> - Context.MODE_PRIVATE :
>   默认操作模式，代表该文件是私有数据，只能被应用本身访问，在该模式下，写入的内容会覆盖原文件的内容，如果想把新写入的内容追加到原文件中。可以使用Context.MODE_APPEND
> - Context.MODE_APPEND：模式会检查文件是否存在，存在就往文件追加内容，否则就创建新文件。
> - MODE_WORLD_READABLE：表示当前文件可以被其他应用读取；
> - MODE_WORLD_WRITEABLE：表示当前文件可以被其他应用写入。

>- 如果希望文件被其他应用读和写，可以传入： 
>- openFileOutput("itcast.txt", Context.MODE_WORLD_READABLE + Context.MODE_WORLD_WRITEABLE);
>- android有一套自己的安全模型，当应用程序(.apk)在安装时系统就会分配给他一个userid，当该应用要去访问其他资源比如文件的时候，就需要userid匹配。默认情况下，任何应用创建的文件，
>- sharedpreferences，数据库都应该是私有的（位于/data/data/<package name>/files），其他程序无法访问。除非在创建时指定了Context.MODE_WORLD_READABLE或者
>- Context.MODE_WORLD_WRITEABLE ，只有这样其他程序才能正确访问。

### 读取文件内容 ###
* 如果要打开存放在/data/data/<package name>/files目录应用私有的文件，可以使用Activity提供openFileInput()方法。

  `FileInputStream inStream = this.getContext().openFileInput("itcast.txt");`

* 直接使用文件的绝对路径：

      File file = new File("/data/data/com.suixin/files/Susion.txt");
    	FileInputStream inStream = new FileInputStream(file);
    	对于私有文件只能被创建该文件的应用访问，如果希望文件能被其他应用读和写，可以在创建文件时，指定Context.MODE_WORLD_READABLE和Context.MODE_WORLD_WRITEABLE权限。

### SD卡与手机内存路径的获取 ###
* 获取SD卡路径：

      File sdCardDir = Environment.getExternalStorageDirectory();   
    	or
    	File sdCardDir = new File("/mnt/sdcard"); //获取SDCard目录

->为何要获取SDcard路径：

>   用Activity的openFileOutput()方法保存文件，文件是存放在手机空间上，一般手机的存储空间不是很大，存放些小文件还行，如果要存放像视频这样的大文件，是不可行的。对于像视频这样的大文件，我们可以把它存放在SDCard。 

#### 把文件放在SDcard中 ####

>- 要往SDCard存放文件，程序必须先判断手机是否装有SDCard，并且可以进行读写（还要注意是否有权限）。

	if(Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)){
	         File sdCardDir = Environment.getExternalStorageDirectory();//获取SDCard目录
	         File saveFile = new File(sdCardDir, “susion.txt”);
		FileOutputStream outStream = new FileOutputStream(saveFile);
		outStream.write("一键恢复".getBytes());
		outStream.close();
	}

>- Environment.getExternalStorageState()方法用于获取SDCard的状态，如果手机装有SDCard，并且可以进行读写，那么方法返回的状态等于Environment.MEDIA_MOUNTED。

>- NT:在程序中访问SDCard，你需要申请访问SDCard的权限。
>    - android.permission.MOUNT_UNMOUNT_FILESYSTEMS  // 挂载和卸载SDCard
>    - android.permission.WRITE_EXTERNAL_STORAGE // 写入外存储设备权限

* 获取手机内存路径：

  `File memoryDir = Environment.getDataDirectory();`

## SharedPreferences ##
### 使用SharedPreferences进行数据存储 ###
> - 很多时候我们开发的软件需要向用户提供软件参数设置功能，例如我们常用的QQ，用户可以设置是否允许陌生人添加自己为好友。
> - 对于软件配置参数的保存，如果是window软件通常我们会采用ini文件进行保存，如果是j2se应用，我们会采用properties属性文件或者xml进行保存。
> - 如果是Android应用，我们最适合采用什么方式保存软件配置参数呢？Android平台给我们提供了一个SharedPreferences类，它是一个轻量级的存储类，
> - 特别适合用于保存软件配置参数。使用SharedPreferences保存数据，其背后是用xml文件存放数据，文件存放在/data/data/<package name>/shared_prefs目录下。

使用方法：

	SharedPreferences sharedPreferences = getSharedPreferences("suixin", Context.MODE_PRIVATE);
	Editor editor = sharedPreferences.edit();//获取编辑器
	editor.putString("name", "吕子乔");
	editor.putInt("age", 28);
	editor.commit();//提交修改

生成的suixin.xml文件内容如下：

	<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
	<map>
		<string name="name">吕子乔</string>
		<int name="age" value="28" />
	</map>
	
	因为SharedPreferences背后是使用xml文件保存数据，getSharedPreferences(name,mode)方法的第一个参数用于指定该文件的名称，名称不用带后缀，后缀会由Android自动加上。
	方法的第二个参数指定文件的操作模式，共有四种操作模式，这四种模式上面有。如果希望SharedPreferences背后使用的xml文件能被其他应用读和写，
	可以指定Context.MODE_WORLD_READABLE和Context.MODE_WORLD_WRITEABLE权限。
	另外Activity还提供了另一个getPreferences(mode)方法操作SharedPreferences，这个方法默认使用当前类不带包名的类名作为文件的名称。

### 访问SharedPreferences中的数据 ###
	SharedPreferences sharedPreferences = getSharedPreferences("suixin", Context.MODE_WORLD_READABLE);
	//do somethings

#### 访问其他应用中的Preference ####
有两个前提条件是：
> - 两个应用程序需要在AndroidManifest.xml中manifest节点里添加sharedUserId属性，并且要一样，而且还要有两级，也就是需要有个“.”
> - 该preference创建时必须指定了Context.MODE_WORLD_READABLE或者Context.MODE_WORLD_WRITEABLE权限。

满足上面两个条件之后在我们应用中需要得到另一个应用的上下文对象：

	Context otherAppsContext = createPackageContext("com.suixin.action", Context.CONTEXT_IGNORE_SECURITY);

	拿到上下文对象之后通过调用上下文中的方法得到SharedPreferences对象，最后进行数据的交互。
	SharedPreferences sharedPreferences = otherAppsContext.getSharedPreferences("suixin", Context.MODE_WORLD_READABLE);
	String name = sharedPreferences.getString("name", "");  //getString()第二个参数为缺省值，如果preference中不存在该key，将返回缺省值
	int age = sharedPreferences.getInt("age", -1);

