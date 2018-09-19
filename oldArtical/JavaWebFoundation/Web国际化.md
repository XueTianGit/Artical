title: JavaWebFoundation- Web国际化
date: 2016/3/1 9:18:40               
categories: JavaWebFoundation
---


# Web国际化 #

1. 软件的国际化：软件开发时，要使它能同时应对世界不同地区和国家的访问，并针对不同地区和国家的访问，提供相应的、符合来访者阅读习惯的页面或数据。
2. 国际化又称为 i18n：internationalization
3. 应分别对固定数据、动态数据进行国际化

## 固定文本元素的国际化 ##
* 对于软件中的菜单栏、导航栏等，可以把他们编写在一个properties文件中，并根据不同国家编写不同的properties文件，这样的一组properties文件称为一个资源包。
* 在JavaAPI中提供了一个ResourceBundle 类用于描述一个资源包，并且 ResourceBundle类提供了相应的方法getBundle，这个方法可以根据来访者的国家地区自动获取与之对应的资源文件予以显示。

### 创建资源包和资源文件 ###
* 一个资源包中的每个资源文件都必须拥有共同的基名。除了基名，每个资源文件的名称中还必须有标识其本地信息的附加部分。
*  每个资源包都应有一个默认资源文件，这个文件不带有标识本地信息的附加部分。若ResourceBundle对象在资源包中找不到与用户匹配的资源文件，它将选择该资源包中与用户最相近的资源文件，如果再找不         	到，则使用默认资源文件
   NT：属性文件不能保存中文资源文件的书写格式
*  资源文件的内容通常采用“关键字＝值”的形式，软件根据关键字检索值显示在页面上。一个资源包中的所有资源文件的关键字必须相同，值则为相应国家的文字
> 	
	//范例：
	//ResourceBundle bundle=ResourceBundle.getBundle("com.hbsi.resource.MyResource");
	ResourceBundle bundle=ResourceBundle.getBundle("com.hbsi.resource.MyResource", Locale.ENGLISH);   //（基名， 国家）
	String username=bundle.getString("username");
    String password=bundle.getString("password");
    String submit=bundle.getString("submit");
    System.out.println(username+":"+password+":"+submit);


> NT： 并且资源文件中采用的是properties格式文件，所以文件中的所有字符都必须是ASCII字码，对于像中文这样的非ACSII字符，须先进行编码。(java提供了一个native2ascii命令用于编码)。

## 动态数据的国际化 ##
> 数值，货币，时间，日期等数据由于可能在程序运行时动态产生，所以无法像文字一样简单地将它们从应用程序中分离出来，
>   而是需要特殊处理。Java 中提供了解决这些问题的 API 类(位于java.util 包和java.text 包中)

- Locale 类
   - Locale实例对象代表一个特定的地理，政治、文化区域。
   - 一个 Locale 对象本身不会验证它代表的语言和国家地区信息是否正确，只是向本地敏感的类提供国家地区信息，与国际化相关的格式化和解析任务由本地敏感的类去完成。
  (若JDK中的某个类在运行时需要根据 Locale 对象来调整其功能，这个类就称为本地敏感类)

- DateFormat类
   - DateFormat 对象的方法：
>
	format： 将date对象格式化为符合某个本地环境习惯的字符串。
	parse：将字符串解析为日期/时间对象
	NT：parse和format完全相反，一个是把date时间转化为相应地区和国家的显示样式，一个是把相应地区的时间日期转化成date对象，
    该方法在使用时，解析的时间或日期要符合指定的国家、地区格式，否则会抛异常。
	//反向解析--字符串要满足构建的格式字符串的格式,否则抛异常
	DateFormat df=DateFormat.getDateInstance(DateFormat.FULL,Locale.CHINESE);
    String str="2011年11月14日 星期一";
    Date d1=df.parse(str);
    System.out.println(d1);

> NT：DateFormat 对象通常不是线程安全的，每个线程都应该创建自己的 DateFormat 实例对象
 
- NumberFormat类
  - NumberFormat 可以将一个数值格式化为符合某个国家地区习惯的数值字符串，也可以将符合某个国家地区习惯的数值字符串解析为对应的数值
>
	NumberFormat 类的方法：
	format 方法：将一个数值格式化为符合某个国家地区习惯的数值字符串
	parse 方法：将符合某个国家地区习惯的数值字符串解析为对应的数值。

	实例化NumberFormat类时，可以使用locale对象作为参数，也可以不使用，下面列出的是使用参数的。
	getNumberInstance(Locale locale)：以参数locale对象标识的本地信息来获得具有**多种用途**的NumberFormat实例对象
	getIntegerInstance(Locale locale)：以参数locale对象所标识的本地信息来获得处理**整数**的NumberFormat实例对象
	getCurrencyInstance(Locale locale)：以参数locale对象所标识的本地信息来获得处理**货币**的NumberFormat实例对象
	getPercentInstance(Locale locale)：以参数locale对象所标识的本地信息来获得处理**百分比数值**的NumberFormat实例对象

	//货币数据的国际化————getCurrencyInstance()可以带参数，如果带参数，调用的是国家名，设置的是数字前面的￥、$等符号   
 	NumberFormat nf=NumberFormat.getCurrencyInstance(Locale.US);
       double f=56.8;
       str=nf.format(f);
       System.out.println(str);