title: JavaWebFoundation- 将JavaBean对象/List或Set或Map对象转成JSON
date: 2016/3/1 9:18:40               
categories: JavaWebFoundation
---

# 将JavaBean对象/List或Set或Map对象转成JSON #


## 使用Struts内置功能进行转换 ##
步骤：

- 导入 struts2-json-plug.jar
- 在struts.xml中，让我们自定义的包继承“json-default”:
> 
	<package name="myPackage" extends="json-default" namespace="/">
	</package>
- 在Action的<result/>中添加json返回类型
> 	
	<result name="success" type="json"/>
- **为将要转换的对象提供get方法**

	在jsp页面中，对于转换的结果我们可以这样获取
	var jsonJAVA = ajax.responseText;	  //注意，我们获取到的是JSON类型的字符串，也可说jsonJAVA
	NT:java格式的json文本，是不能直接被js执行的
	->  解决方案：将java格式的json文本，转成js格式的json文本, 使用eval()函数，但应注意书写格式，即变量要加上"()"
		var jsonJS = eval("("+jsonJAVA+")");

## 使用第三方工具进行转换 ##
	1. 准备导入第三方jar包：
	    》commons-beanutils-1.7.0.jar
	    》commons-collections-3.1.jar
	    》commons-lang-2.5.jar
		》commons-logging-1.1.1.jar
		》ezmorph-1.0.3.jar
		》json-lib-2.1-jdk15.jar		
	
	2.使用API进行，转换，并将结果以流的形式，打给AJAX
	（1）JavaBean----->JSON
		》JSONArray jsonArray = JSONArray.fromObject(city);
		》String jsonJAVA = jsonArray.toString();
	（2）List<JavaBean>----->JSON 
	    》JSONArray jsonArray = JSONArray.fromObject(cityList);
	    》String jsonJAVA = jsonArray.toString();
	（3）List<String>----->JSON 
	    》JSONArray jsonArray = JSONArray.fromObject(stringList);
	    》String jsonJAVA = jsonArray.toString(); 
	（4）Set<JavaBean>----->JSON 
	    》JSONArray jsonArray = JSONArray.fromObject(citySet);
	    》String jsonJAVA = jsonArray.toString();
	（5）Map<String,Object>----->JSON 
	    》JSONArray jsonArray = JSONArray.fromObject(map);
	    》String jsonJAVA = jsonArray.toString();

