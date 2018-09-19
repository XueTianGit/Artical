title: JQuery- AJAX相关的API
date: 2016/2/29 19:14:41           
categories: WebFrontEnd
---

# JQuery- AJAX相关的API#

> 目的：简化客户端与服务端进行局部刷新的异步通讯。（简化编写AJAX代码）。

## load() ##
> 简单形式：$.load(url)	

	返回结果自动添加到jQuery对象代表的标签中间
	如果是Servlet的话，采用的是GET方式

> 复杂形式：$.load(url,sendData,function(backData,textStatus,ajax){... ...})

	请求方式：
	->如果请求体无参数发送的话，load方法采用GET方式提交
	->如果请求体有参数发送的话，load方法采用POST方式提交, 并且会自动进行编码，无需手工编码
	
	参数解释：
    参数一： url发送到哪里去
    参数二： sendData发送请求体中的数据，符合JSON格式，例如：{key:value,key:value}
    参数三： function处理函数，类似于传统方式ajax.onreadystatechange = 处理函数

    其中参数三为function处理函数最多可以接收三个参数，含义如下
	第一个参数：服务端返回的数据，例如：backData
	第二个参数：服务端状态码的文本描述，例如：success、error、
	第三个参数：ajax异步对象，即XMLHttpRequest对象
    以上所有参数的名字可以任意，但必须按顺序书写，尽量做到见名知意

范例代码：

	<script type="text/javascript">
		$(":button").click(function(){
			var url = "${pageContext.request.contextPath}/loadTimeRequest?time"+new Date().getTime();
			var sendData = null;
			$.load(url,sendData,function(backData,textStatus,ajax){
				/*doSomething*/
			});
		});
	</script>

## get()与post() ##

使用方式：

	$.get(url,sendData,function(backData,textStatus,ajax){... ...})
	$.post(url,sendData,function(backData,textStatus,ajax){... ...})

> 参数含义与load相同，并且使用get或post方法时，自动进行编码，无需手工编码

## ajax ##

	$.ajax的一般格式:
	
	$.ajax({
	    type: 'POST',
	    url: url ,
	    data: data ,
	    success: success ,
	    dataType: dataType
	});
	
	参数：
		type     提交的类型
		url	     必需。规定把请求发送到哪个 URL。
		data	 可选。映射或字符串值。规定连同请求发送到服务器的数据。
		success(data, textStatus, jqXHR)	可选。请求成功时执行的回调函数。
		dataType	可选。规定预期的服务器响应的数据类型。默认执行智能判断（xml、json、script 或 html）。
	注意：
	  1.data主要方式有三种，html拼接的，json数组，form表单经serialize()序列化的；不指定智能判断。
	  2.$.ajax只提交form以文本方式，如果异步提交包含<file>上传是传过不过去,需要使用jquery.form.js的$.ajaxSubmit

## 使用serialize()自动生成JSON文本 ##

	jQuery对象.serialize()：
	作用：自动生成JSON格式的文本
	注意：为每个jQuery对象设置一个name属性，因为name属性会被认为请求参数名
	注意：必须用<form>标签元素套上
	适用：如果属性过多，强烈推荐采用这个API 
	
		<script type="text/javascript">
			//定位按钮，同时添加单击事件
			$(":button").click(function(){
				//获取用户名和密码
				var username = $(":text:first").val();
				var password = $(":text:last").val();
				//去空格
				username = $.trim(username);
				password = $.trim(password);
				//异步发送到服务端
				var url = "${pageContext.request.contextPath}/checkRequest?time="+new Date().getTime();
				/*手工书写JSON文本
				var sendData = {
					"user.username":username,
					"user.password":password
				};
	
				*/
				/*工具生成JSON文本*/
				var sendData = $("form").serialize();
		</script>
        

