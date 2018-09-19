title: JavaWebFoundation- EL表达式的主要作用
date: 2016/3/1 9:18:40               
categories: JavaWebFoundation
---

# EL表达式的主要作用   #

使语法：可以直接在JSP页面中 ->  ${expression}

## 获取数据 ##

* 获取域中的数据（pageContext、Request、Session、applicationScope --> 实际上底层是使用的pageContext.findAttribute()方法， 依次从这四个域查找）
* 获取域中存在的bean中的数据  -> ${user.name}
* 获取域中存的List集合的数据
* 获取域中存的Map集合的数据

--> 总之，就是用来获取数据的。 
NT：**EL表达式有一个特点，即当获取的数据不存在时，返回的是""** ;

###  . 与 [] ###
EL 提供.和[]两种运算符来获取数据。不过主要使用 . 来获取数据
 
* **何时用 [] ?**
当要存取的属性名称中包含一些特殊字符，如.或?等并非字母或数字的符号，就一定要使用 [] 。
example:		${user.My-Name}应当改为${user["My-Name"] }

如果要动态取值时，就可以用[]来做，而.无法做到动态取值。例如： ${sessionScope.user[data]}, 这里data 是一个变量

## 执行运算 ##
在EL表达式中支持基本算术运算符、关系运算符和基本逻辑运算符。当然算术运算符返回的是算术结果，逻辑运算符返回的是true 或者 false 。
example： ${5 + 5} --->10 ; ${1 > 2} ---> false

--> 奇特点的：
empty运算符： 
用法：${empty user} 用于判断对象是否为null。
三目运算符：
三目运算符，一直就很强。 在EL中一个特别的应用是数据回显: 	<input type="radio" name="qender" value="female" ${qender=='female' ? checked：'' } />

## 获取Web开发中常用对象 ##
EL总共定义了11个隐式对象。

### pageContext ###
利用pageContext，我们可以的得到request、session什么的。 例如获取Web应用的根目录 ： ${pageContext.request.contextPath}

### 与范围有关的4个 ###
pageScope、requestScope、sessionScope 和 applicationScope；
EL表达式获取数据就是从这里获取。  ${requestScope.name} <---> request.getAttribute("name");

### 与输入有关的隐含对象 ###
param、paramValues
param类似一个封装了所有请求参数的map集合，（url参数， 表单数据）
paramvalues和param类似， 只不过该map集合的值是一个数组。 例如  肉--->猪肉、鸡肉（一对多）

### 其他隐式对象 ###
cookie： 一个保存了所有cookie的map对象  S{cookie.JSESSION.name}
header与headerValues: 表示一个保存所有http请求头的Map集合对象。 ${header.Host}
initParam： 保存了所有web应用初始化参数的Map对象。

## 调用Java方法 ##
EL表达式允许开发人员自定义函数，以调用java方法。
步骤：
1. 在EL表达式中调用的只能是java类的静态方法。 
	public class MyEL {
	//el函数
	public static String filter(String message) {
	}
	}

2. 这个java类的静态方法需要在tld文件中描述，才可以被EL表达式调用。
	<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-jsptaglibrary_2_0.xsd"
    version="2.0">
	
	<tlib-version>1.0</tlib-version>
    	<short-name>SimpleTagLibrary</short-name>
    <uri>/my</uri>
    
    <function>
	    <name>filter</name>
		<function-class>cn.itcast.demo.MyEL</function-class>
		<function-signature>java.lang.String filter( java.lang.String )</function-signature>
    </function>

	</taglib>
3. 使用
	-> 导入标签库    <%@taglib uri="/my" prefix="my" %>
	-> 使用 			${my:filter("<a href=''>点点</a>") }

NT :** EL函数可以移除Java代码， 但他是代替不了自定义标签的， 因为他无法移除与web开发相关的代码**

# Sun公司的EL函数库  #

这些函数主要是针对字符串显示的一些函数。
NT：使用这些函数前必须引入JSTL开发包， 并在页面中引入EL函数库 （ <%@taglib uri="http://java.sun.com/jsp/jstl/functions" prefix="fn" %> ） 
并且Sun公司所有的EL函数都在fn.tld 文件中描述的。

函数列表：

1.  fn:contains 判断字符串是否包含另外一个字符串 <c:iftest="${fn:contains(name,searchString)}">   
2.  fn:containsIgnoreCase 判断字符串是否包含另外一个字符串(大小写无关) 
    <c:iftest="${fn:containsIgnoreCase(name,searchString)}">   
3.  fn:endsWith 判断字符串是否以另外字符串结束 <c:iftest="${fn:endsWith(filename,".txt")}">   
4.  fn:escapeXml 把一些字符转成XML表示，例如<字符应该转为&lt;    ${fn:escapeXml(param:info)}   
5.  fn:indexOf 子字符串在母字符串中出现的位置 ${fn:indexOf(name,"-")}   
6.  fn:join 将数组中的数据联合成一个新字符串，并使用指定字符格开 ${fn:join(array,";")}   
7.  fn:length 获取字符串的长度，或者数组的大小 ${fn:length(shoppingCart.products)}   
8.  fn:replace 替换字符串中指定的字符 ${fn:replace(text, "-","&#149;")}   
9.  fn:split 把字符串按照指定字符切分 ${fn:split(customerNames,";")}   
10. fn:startsWith 判断字符串是否以某个子串开始 <c:iftest="${fn:startsWith(product.id,"100-")}">   
11. fn:substring 获取子串  ${fn:substring(zip, 6,-1)}   
12. fn:substringAfter 获取从某个字符所在位置开始的子串 ${fn:substringAfter(zip,"-")}   
13. fn:substringBefore 获取从开始到某个字符所在位置的子串 ${fn:substringBefore(zip,"-")}   
14. fn:toLowerCase 转为小写${fn.toLowerCase(product.name)}   
15. fn:toUpperCase 转为大写字符${fn.UpperCase(product.name)}   
16. fn:trim 去除字符串前后的空格 ${fn.trim(name)} 









