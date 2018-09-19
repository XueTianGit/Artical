title: JavaScript基础回顾
date: 2016/2/29 19:14:41           
categories: WebFrontEnd
---

# JavaScript基础回顾 #

## 什么是JavaScript ##
- 基于对象
>      JS本身就有一些现成的对象可供程序员使用，例如：Array，Math，String。。。
>      JS并不排除你可以自已按一定的规则创建对象

- 事件驱动
>      JS代码写好后，需要外界触发后，方可运行，例如：单击事件，定时执行，。。。

- 解释性
> 	 每次运行JS代码时，得需要将原代码一行一行的解释执行
>      相对编译型语言（例如：Java、C++）执行速度相对较慢

- 基于浏览器的动态交互网页技术
> 	 如果JS嵌入到HTML中，可以不需要服务器支持，直接由浏览器解释执行
> 	 如果JS嵌入到JSP或Servlet中，必需要服务器支持，直接由浏览器解释执行

- 嵌入在HTML标签中
> 	JS必须嵌入到一个名叫<script src="引入外部js文件"></script>的标签中，方可运行。

- 弱类型语言
> 	var关键字就是代表，但弱类型不代表没有类型

## JS中的三种类型 ##
	（1）基本类型：number，string，boolean
	     number包含正数，负数，小数
		 string由''或""定界
		 boolean由true或false，但js中的boolean也包含更多情况，例如：存在表示true，不存在表示false
	（2）特殊类型：null，undefined
		 null表示一个变量指向null
		 undefined表示一个变量指向的值不确定
		 -> 判断某个变量是否为undefined
				//undefined不是字符串，它是一种类型
				 if(card == undefined){
					alert("card变量暂没值");		 
				 }else{
				 	alert(card);
				 }
	（3）复合类型：函数，对象，数组
		    对象包含内置对象和自定义的对象

## JS中定义函数的三种形式 ##
	（1）正常方式：
	        function mysum(num1,num2){
				return num1 + num2;
			}
			var myresult = mysum(100,200);
			alert("myresult="+myresult);	
	（2）构造器方式：
			var youresult = new Function("num1","num2","return num1+num2");   //这样写也可以“num1， num2”
			alert( youresult(1000,2000) );
	（3）直接量或匿名或无名方式：
			var theyresult = function(num1,num2){
								return num1 + num2;	
							 }
			alert( theyresult(10000,20000) );

## JS中四种对象 ##
	（1）内置对象 ：Date，Math，String，Array，。。。
	（2）自定义对象：Person，Card，。。。
		function Student(id,name,sal){
				//this指向s引用
				this.id = id;
				this.name = name;
				this.sal = sal;
				this.show = function(){
					alert("show()");
				}
			}
			var s = new Student(1,"波波",7000);
			document.write("编号:" + s.id + "<br/>");
			document.write("姓名:" + s.name + "<br/>");
			document.write("薪水:" + s.sal + "<br/>");
	（3）浏览器对象： window，document，status(左下角那个东西吧)，location（地址栏），history。。。
		function myrefresh(){
			window.history.go(0);
		}
	（4）ActiveX对象：ActiveXObject("Microsoft.XMLHTTP")，。。。

## JS对象的属性，方法和事件的使用 ##
	（1）window.location.href
			var url = "04_array.html";
			window.location.href = url;	    //相当于在地址栏敲了个地址
	（2）form.submit()   
		//点击按钮， 表单提交                 
		<form action="/js-day01/04_array.html" method="POST">
			<input type="button" value="提交到服务端" onclick="doSubmit()"/>
		</form>
		<script type="text/javascript">
			function doSubmit(){
				//表单提交
				document.forms[0].submit();
			}
		</script>	 


NT：本博文参考自传智博客课堂笔记