title: Android-基础扫尾
date: 2016/2/29 15:40:29   
categories: Android
---

# Android-基础扫尾 #

>- Android基础和JavaEE我是一块学的（双线练兵）。 主要是以javaEE为主， 所以分配给Android基础的时间很少。所以Android基础学的
>- 很慢，不过怎么说呢， 滴水还穿石呢！ 再拖拉， 也终于扫了一遍， 这篇博文，把笔记内容收拾一下， 就可以开始做一些Android项目了，
>- 接下来android时间不会再是滴水穿石了， 而是开始飞流直下了。 哈哈哈哈。。。。吹牛的，，，，，， 


## 简易的自定义控件 ##
- 基本步骤：

1）继承View对象，

	当然，在这里我们也可以继承View的子对象， 不过扩展性就小了， 主要还是看你开发什么组件了，
	反正你的老祖组织肯定是View吧。

2）重写3个构造方法。放在这里再说。  重写onDraw()方法。

	我们自定义组件，有一部分原因是因为原来组件长的太丑了吧， 那么我们就可以在onDraw()按你的审美观画了。
3）在布局文件中使用我们自定义的组件

	NT:默认情况下， onDraw()方法在Activity显示是会调用一次。若想让再次调用， 则可以使用invalidate()方法
	invalidate()的原理是：是控件无效， 但这时如果控件仍然获得用户焦点的话， 那么就会再次调用onDraw()方法。

代码范例：

	public class MyView extends View {
		public MyView(Context context, AttributeSet attrs, int defStyle) {
			super(context, attrs, defStyle);
			// TODO Auto-generated constructor stub
		}
		public MyView(Context context, AttributeSet attrs) {
			super(context, attrs);
			// TODO Auto-generated constructor stub
		}
		public MyView(Context context) {
			super(context);
			// TODO Auto-generated constructor stub
		}
	
		//在组件里绘制内容
		@Override
		protected void onDraw(Canvas canvas) {
			
			Paint paint = new Paint();
			paint.setColor(Color.RED);
			paint.setTextSize(20);	
			canvas.drawText("简易自定义控件", 10, 20, paint);
			canvas.drawRect(50, 100, 70, 200, paint);	
		}		
	}
	
	    <com.suixin.customview.MyView
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        />


## SharedPreferences ##
>- 很多时候我们开发的软件需要向用户提供软件参数设置功能，例如我们常用的QQ，用户可以设置是否允许陌生人添加自己为好友。对于软件配置参数的保存，
>- 如果是window软件通常我们会采用ini文件进行保存，如果是j2se应用，我们会采用properties属性文件或者xml进行保存。


> - 如果是Android应用，我们最适合采用什么方式保存软件配置参数呢？
> - Android平台给我们提供了一个SharedPreferences类，它是一个轻量级的存储类，特别适合用于保存软件配置参数。
> - 使用SharedPreferences保存数据，其背后是用xml文件存放数据，文件存放在/data/data/<package name>/shared_prefs目录下。

> 例如在Activity（Context的子类）中，我们可以这样使用SharedPreferences：

	//写     
	SharedPreferences sharedPreferences = getSharedPreferences("suixin", Context.MODE_PRIVATE);    //没有即创建 suixin.xml
	Editor editor = sharedPreferences.edit();//获取编辑器
	editor.putString("name", "大黄蜂");
	editor.putInt("age", 500);
	editor.commit();//提交修改
	//读
	String name = sharedPreferences.getString("name");

	生成的suixin.xml文件内容如下：
	<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
	<map>
	<string name="name">传智播客</string>
	<int name="age" value="4" />
	</map>


> - 因为SharedPreferences背后是使用xml文件保存数据，
> - getSharedPreferences(name,mode)方法的第一个参数用于指定该文件的名称，名称不用带后缀，后缀会由Android自动加上。
> 方法的第二个参数指定文件的操作模式，共有四种操作模式，

文件的四种操作模式：

	Context.MODE_PRIVATE：为默认操作模式，代表该文件是私有数据，只能被应用本身访问，在该模式下，写入的内容会覆盖原文件的内容。
	Context.MODE_APPEND：模式会检查文件是否存在，存在就往文件追加内容，否则就创建新文件。
	
	
	MODE_WORLD_READABLE和MODE_WORLD_WRITEABLE用来控制其他应用是否有权限读写该文件：
	Context.MODE_WORLD_READABLE：表示当前文件可以被其他应用读取；
	Context.MODE_WORLD_WRITEABLE：表示当前文件可以被其他应用写入。

2）另外Activity还提供了另一个getPreferences(mode)方法操作SharedPreferences，这个方法默认使用当前类不带包名的类名作为文件的名称。
   一旦我们都使用上一种。

## XmlSerializer与XmlPullParser ##
>- XML文件的生成方式有很多种， 上面不就是一种吗。不过在androif中使用XmlSerializer去生成XML文件，确实是一种不错的选择。
>- XmlPullParser（基于事件）去解析XML文件，也很简单。


生成Xml数据:

	public static String writeXML(List<Person> persons, Writer writer){
	    XmlSerializer serializer = Xml.newSerializer();    //按XML文档的格式生成XML文件
	    try {
	        serializer.setOutput(writer);
	        serializer.startDocument("UTF-8", true);
	        serializer.startTag("", "persons");        //第一个参数为命名空间,如果不使用命名空间,可以设置为null
	        for (Person person : persons){
	            serializer.startTag("", "person");
	            serializer.attribute("", "id", person.getId().toString());//设置属性
		            serializer.startTag("", "name");
		            serializer.text(person.getName());
		            serializer.endTag("", "name");
		            serializer.startTag("", "age");
		            serializer.text(person.getAge().toString());
		            serializer.endTag("", "age");
	            serializer.endTag("", "person");
	        }
	        serializer.endTag("", "persons");
	        serializer.endDocument();
	        return writer.toString();
	    } catch (Exception e) {
	        e.printStackTrace();
	    }
	    return null;
	}



解析Xml文件:
	
	public static List<Person> readXML(InputStream inStream) {
		XmlPullParser parser = Xml.newPullParser();
		try {
		parser.setInput(inStream, "UTF-8");
		int eventType = parser.getEventType();
		Person currentPerson = null;
		List<Person> persons = null;
		while (eventType != XmlPullParser.END_DOCUMENT) {
			switch (eventType) {
				case XmlPullParser.START_DOCUMENT://文档开始事件,可以进行数据初始化处理
					persons = new ArrayList<Person>();        
					break;
				case XmlPullParser.START_TAG://开始元素事件
					String name = parser.getName();    
					if (name.equalsIgnoreCase("person")) {  //判断是否使我们需要的元素person
						currentPerson = new Person();
						currentPerson.setId(new Integer(parser.getAttributeValue(null, "id")));
					} else if (currentPerson != null) {
						if (name.equalsIgnoreCase("name")) {
							currentPerson.setName(parser.nextText());// 如果后面是Text节点,即返回它的值
						} else if (name.equalsIgnoreCase("age")) {
							currentPerson.setAge(new Short(parser.nextText()));
						}
				}
				break;
			case XmlPullParser.END_TAG://结束元素事件
				if (parser.getName().equalsIgnoreCase("person") && currentPerson != null) {
					persons.add(currentPerson);
					currentPerson = null;
				}
				break;
			}
			eventType = parser.next();
		}
		inStream.close();
		return persons;
		} catch (Exception e) {
			e.printStackTrace();
		}
		return null;
	}


对应的XML文件：

	<?xml version="1.0" encoding="UTF-8"?>
	<persons>
		<person id=“18">
			<name>allen</name>
			<age>36</age>
		</person>
		<person id=“28">
			<name>james</name>
			<age>25</age>
		</person>
	</persons>



