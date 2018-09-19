title: jQuery的简单了解
date: 2016/2/29 19:17:16               
categories: WebFrontEnd
---

# jQuery的简单了解 #


## 什么是jQuery ##
> 它是John Resig在2006年1月发布的一款跨主流浏览器的**JavaScript库**，简化JavaScript对HTML操作。

## 为什么要使用jQuery ##
> - （1）写少代码,做多事情【write less do more】
> - （2）免费，开源且轻量级的js库，容量很小
>       注意：项目中，提倡引用min版的js库 ->什么是迷你版呢，看看就知道了！就是把空格、注释都去掉，文件变小了，就mini了。。。。。。
> - （3）兼容市面上主流浏览器，例如 IE，Firefox，Chrome
>      注意：jQuery不是将所有JS全部封装，只是有选择的封装
> - （4）能够处理HTML/JSP/XML、CSS、DOM、事件、实现动画效果，也能提供异步AJAX功能
> - （5）文档手册很全，很详细
> - （6）成熟的插件可供选择
> - （7）提倡对主要的html标签提供一个id属性，但不是必须的
> - （8）出错后，有一定的提示信息
> - （9）不用再在html里面通过`<script>`标签插入一大堆js来调用命令了

## jQuery开发步骤 ##

	（1）引用第三方js库文件
		`<script type="text/javascript" src="js/jquery-1.8.2.js"></script>`
	（2）查阅并使用api手册，
	范例：
	
	//var divElement = document.getElementById("divID");  <--> var $div = $("#divID");
	//var html = divElement.innerHTML;  <-->  $div.html();

## js对象和jQuery对象相互转换 ##

> 什么是js对象及代码规则

	就是使用js-API，即Node接口中的API或是传统JS语法定义的对象，叫做js对象
	js代码规则----divElement
	var divElement = document.getElementById("divID");
	var nameArray = new Array(3);
	
>什么是jQuery对象及代码规则

	就是使用jQuery-API，返回的对象就叫做jQuery对象
	jQuery代码规则----$div
	var $div = $("#divID")
	声明：上述代码规则，只是老师个人规则，不代表所有企业都这样做
	
>js对象转成jQuery对象【重点】

	语法：$(js对象)---->jQuery对象
	例如：$(divElement)---->$div
	例如：$(this)---->$this
	注意：jQuery对象将js对象做了封装，js对象二边无引号
	
	var inputElement = document.getElementById("inputID");//js对象 
	var $input = $(inputElement);//jquery对象
	var txt = $input.val();
	alert(txt);	 
	
> jQuery对象转成js对象

	语法1：jQuery对象[下标，从0开始]
	语法2：jQuery对象.get(下标，从0开始)
	例如：$div[0]---->divElement
	注意：不同的对象只能调用对应的api方法，即jQuery对象不能调用js对象的api，反之亦然
	$div.innerHTML（错）
	divElement.html(错) 
	
	var $div = $("#divID");//jquery对象
	var divElement = $div[0];//js对象(方式一)
	//var divElement = $div.get(0);//js对象(方式二)
	var txt = divElement.innerHTML;		  
	alert(txt);	 
	
	
## js对象和jQuery对象的区别 ##

> js对象的三种基本定位方式

	（A）通过ID属性：document.getElementById()
	（B）通过NAME属性：document.getElementsByName()
	（C）通过标签名：document.getElementsByTagName()
>jQuery对象的三种基本定位方式

	（A）通过ID属性：$("#id属性值")
	（B）通过标签名：$("标签名")
	（C）通过CLASS属性：$(".样式名")
>js对象出错的显示

	没有合理的提示信息
	例如：alert(document.getElementById("usernameIDD").value)
>jQuery对象出错的显示

	有合理的提示信息，例如：undefined
	例如：alert($("#usernameIDD").val())	


