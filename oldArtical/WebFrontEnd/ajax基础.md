title: AJAX基础
date: 2016/2/29 19:14:14          
categories: WebFrontEnd
---

# AJAX基础 #

## 什么是AJAX ##
全称是： Asynchronous JavaScript And XML

> 客户端（特指PC浏览器）与服务器，可以在【不必刷新整个浏览器】的情况下，与服务器进行异步通讯的技术。即，AJAX是一个【局部刷新】的【异步】通讯技术

NT：AJAX不是全新的语言，是2005年Google公司推出的一种全新【编程模式】，不是新的编程语言

不用刷新整个页面便可与服务器通讯的办法有：
> - Flash/ActionScript
> - 框架Frameset
> - iFrame（内嵌入框架)
> - XMLHttpRequest(非IE浏览器)和ActiveXObject(IE浏览器)

> 背景：早上IE5时，微软就开发出了第一个异步通讯对象，叫ActiveXObject对象，
> Firefox等其它浏览器厂商也慢慢引入异步通讯对象，叫XMLHttpRequest对象，
> IE的高版本，也将这个异步对象取名叫XMLHttpRequest对象，但IE有向下兼容问题，
> 也可以使用ActiveXObject对象。
> 无需第三方jar包，现代中高版本浏览器中内置了这个异步通讯对象，只需通过JavaScript就可以创建
> 
> 注意：所有浏览器中都内置了异步对象，在默认情况下，该异步对象并没有创建出来

### 在使用AJAX时， 相关的技术  ###
XML、HTML、CSS、JS、DOM

### 创建异步对象 ###
    
    function createAJAX(){
    	var ajax = null;
    	try{
    		ajax = new ActiveXObject("microsoft.xmlhttp");
    	}catch(e1){
    		ajax = new XMLHttpRequest();
    	}
    	return ajax;
    }

## AJAX的工作原理 ##

> Ajax的原理简单来说通过XmlHttpRequest对象来向服务器发异步请求，从服务器获得数据，然后用javascript来操作DOM而更新页面。

### XmlHttpRequest对象 ###

>属性：

	readyState： 可取 0、1、2、3、4 分别代表
	0 (未初始化) 对象已建立，但是尚未初始化（尚未调用open方法）
	1 (初始化) 对象已建立，尚未调用send方法
	2 (发送数据) send方法已调用，但是当前的状态及http头未知
	3 (数据传送中) 已接收部分数据，因为响应及http头不全，这时通过responseBody和responseText获取部分数据会出现错误，
	4 (完成) 数据接收完毕,此时可以通过通过responseXml和responseText获取完整的回应数据

	status： 它的值来自与服务器返回的响应状态码（例如 200/404/500）
	responseText ：  从服务器进程返回数据的字符串形式。
	responseXML  ：  从服务器进程返回的DOM兼容的文档数据对象。

>方法
	open(method, url) : method代表提交方式, url是ajax提交的地方。
	
	send(content) ：
	如果是GET的话， 就取null；
	如果是POST的话，contnt为非null，格式为键值对的形式
	
	setRequestHeader("content-type", "application/x-www-form-urlencoded");
	在POST提交方式下必须设置ajax发送的请求头（其实就是设置请求体的编码表）

> 以前我们在使用form表单提交数据时， enctype="application/x-www-form-urlencoded"
> 这是默认的

>事件 
	onreadystatechange: ajax异步对象不断监听服务器状态的变化，当发生变化是就会产生这个事件。
	我们只需使用匿名函数处理这个事件即可
	NT: 这个事件只有在状态改变时才会产生 

## AJAX开发步骤 ##
1. 创建AJAX异步对象，例如：createAJAX()
2. 准备发送异步请求，例如：ajax.open(method,url)
3. 如果是POST请求的话，一定要设置AJAX请求头，例如：ajax.setRequestHeader() 如果是GET请求的话，无需设置设置AJAX请求头
4. 真正发送请求体中的数据到服务器，例如：ajax.send()
5. AJAX不断的监听服务端响应的状态变化，例如：ajax.onreadystatechange，后面写一个无名处理函数	
6. 在无名处理函数中，获取AJAX的数据后，按照DOM规则，用JS语言来操作Web页面	

范例：

	<script type="text/javascript">
		document.getElementsByTagName("input")[0].onclick = function(){
			//NO1)创建AJAX异步对象（每个浏览器内置的，无需第三方jar包）
			var ajax = createAJAX();//0
			//NO2)AJAX异步对象准备发送请求
			var url = "${pageContext.request.contextPath}/TimeServletAjax?id="+new Date().getTime();
			var method = "GET";
			ajax.open(method,url);//1
			//NO3）AJAX异步对象真正发送请求体的数据到服务器，如果请求体无数据的话，用null表示
			var content = null;
			ajax.send(content);//2
			
			//----------------------------------------等待
			
			//NO4）AJAX异步对象不断监听服务端状态的变化，只有状态码变化了，方可触发函数
			//0-1-2-3-4,这些是可以触发函数的
			//4-4-4-4-4，这些是不可以触发函数的
			//以下这个函数是服务器来触发的，不是程序员触发的，这和onclick是不一样的
			ajax.onreadystatechange = function(){
				//如果AJAX状态码为4
				if(ajax.readyState == 4){
					//如果服务器响应码是200
					if(ajax.status == 200){
						//NO5）从AJAX异步对象中获取服务器响应的结果
						var str = ajax.responseText;
						//NO6）按照DOM规则，将结果动态添加到web页面指向的标签中
						/* 处理响应结果*/
					}
				}
			}
		} 
	</script>	 

## AJAX中的数据载体 ##

- HTML

> - 优点：服务端响应的是普通html字符串，无需JS解析，配合innerHTML属性效率高
> - 缺点：如果需要更新WEB页面中的很多处地方，HTML不太方便，同时innerHTML属性不是DOM的标准，不能操作XML,注意：innerHTML在xml中不能使用，用firstChild.nodeValue替代
> - 适合：小量数据载体，且只更新在web页面中的一个地方

- XML
> - 优点：是种通用的普通字符串格式，任何技术都能解析，标签名可以任意，使用DOM标准规则能够解析XML的任何部分
> - 缺点：XML文件格式相当严格，出错后，responseXML属性返回NULL，如果XML文件过长，导致解析效率低下
> - 适合：大量具有层次数据载体     

- JSON 
> - 兼备HTML和XML的特点

