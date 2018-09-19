title: JavaWebFoundation-JSON基础
date: 2016/3/1 9:18:40               
categories: JavaWebFoundation
---

# JSON基础 #

## 什么是JSON  ##
### 概论 ###
* JSON（Java Script Object Notation（记号,标记））是一种轻量级的数据交换语言，以文本字符串为基础，且易于让人阅读
NT：XML就是一个重量级的数据交换语言
* JSON是JavaScript原生格式，这意味着在JavaScript中处理JSON数据不需要任何特殊的API或工具包
* JSON采用完全独立于任何程序语言的文本格式，使JSON成为理想的数据交换语言

> 在服务器端， AJAX 是一门与服务端语言无关的技术，可使用服务器端任何语言。
> AJAX从服务器端接收数据的时候，那些数据必须以浏览器能够理解的格式来发送。服务器端的编程语言只能以如下 3 种格式返回数据：
> HTML、XML 、JSON

## JSON数据格式 ## 
> 对象是一个无序的“‘名称:值’对”集合。一个对象以“{”（左括号）开始，“}”（右括号）结束。每个“名称”后跟一个“:”（冒号）；“‘名称/值’对”之间使用“,”（逗号）分隔。

- 映射用冒号（“：”）表示。名称:值
- 并列的数据之间用逗号（“，”）分隔。名称1:值1,名称2:值2
- 映射的集合（对象）用大括号（“{}”）表示。{名称1:值1,名称2:值2}
- 并列数据的集合（数组）用方括号(“[]”)表示。 
  example：[ {名称1:值,名称2:值2}, {名称1:值,名称2:值2} ]
- 元素值可具有的类型：string, number, object, array, true, false, null 

### 各种利用JSON封装的对象 ###
	example1:
	var p = {id:1, name:'小明', age:18};
	document.write(p.id);
	
	example2:
	var persons = [{id:1, name:'小明', age:18}, {id:2, name:'小王', age:18}, {id:3, name:'小张', age:18}];
	document.write(persons[1].id);
	
	example3:
	var person = {"province":[{city:"BeiJing"},{city:"ShangHai"},{city:"TengZhou"}]};
	document.write(person.province[0].city);
	
	example4:
	var person = {"city":["BeiJing","ShangHai","TengZhou"]};
	document.write(person.city[0]);
	
	example5:
	var p = {
				id:1,
				name:"哈哈",
				tel:[
						{
							no:"135",
							type:"中移动"
						},
						{
							no:"133",
							type:"中联通"
						}
					],
				show:function(username){
					alert("你的姓名是:" + p.name+":"+username);
				},
				isSingle:false			
			};
	
	p.show("Susion");



## JSON的作用 ##
### 简化创建自定义对象的方式 ###
	JSON就是用JS语法来书写，所以必须放在<script>标签中
	具体创建对象的方式，查看上面！

### 在AJAX中，作为数据载体之一 ###

> 注意：JS可以直接解析JSON格式的文本，前提是：该JSON必须采用JS格式书写的才行，如果该JSON是采用Java格式写的，必须使用eval()函数转换后，方可被JS解析，该eval("")函数接收一个字符串格式的内容。


## 解析JSON ##
.......

## JSON小结 ##
* 优点：
	作为一种数据传输格式，JSON 与 XML 很相似，但是它更加轻巧。
	JSON 不需要从服务器端发送含有特定内容类型的首部信息，只是一个html普通字符串而以。

* 缺点：
	语法过于严谨
	代码不易读