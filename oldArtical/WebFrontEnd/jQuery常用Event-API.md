title: jQuery常用Event-API
date: 2016/2/29 19:16:34             
categories: WebFrontEnd
---


# jQuery常用Event-API #
> 目的：对web页面（HTML/JSP）进行事件触发，完成特殊效果的处理


	window.onload：在浏览器加载web页面时触发，可以写多次onload事件，但后者覆盖前者   (这是传统加载内容的办法)
	 
	ready：在浏览器加载web页面时触发，可以写多次ready事件，不会后者覆盖前者，依次从上向下执行，我们常用 $(函数) 简化
	NT： ready和onload同时存在时，二者都会触发执行，ready快于onload
	example: 
			$(function(){
				alert("jQuery加载web页面");
			});
	
	change：当内容改变时触发
	example：
			//当<select>标签触发onchange事件
			$("#selectID").change( function(){ 
				var $option = $("#city option:selected");
				var value = $option.val();
				var text = $option.text();
				alert(value+":"+text);
			} );
	
	focus：焦点获取   

example：

	<input type="text" value="加载页面时获取光标并选中所有文字" size="50"/>	
	<script type="text/javascript">
		//加载页面时获取光标并选中所有文字
		$(function(){
			//光标定位文本框
			$(":text").focus();
			//选中文本框的所有文本;
			$(":text").select()
		});
	</script>

	  blur：焦点失去
	
	
	  select：选中所有的文本值  -> 看上面
	
	  keyup/keydown/keypress：演示在IE和Firefox中获取event对象的不同

example：

			//当按键弹起时，显示所按键的unicode码
			$(function(){
				//IE浏览器会自动创建event这个事件对象，那么程序员可以根据需要来使用该event对象
				$(document).keyup(function(){
					//获取按钮的unicode编码
					var code = event.keyCode; 
					alert(code);
				});
			});
	
	  mousemove：在指定区域中不断移动触发
			$(function(){
				$(document).mousemove(function(){
					//获取当前鼠标的坐标
					var x = event.clientX;
					var y = event.clientY;
		
				});		
			});
	
	
	  mouseover：鼠标移入时触发
			//鼠标移到某行上，某行背景变色
			$("table tr").mouseover(function(){
				$(this).css("background-color","inactivecaption");
			});	
	  mouseout：鼠标移出时触发
	
	 submit：在提交表单时触发，true表示提交到后台，false表示不提交到后台
	->  用于表单提交前检测
		$(form).submit(function(){   });
	
	  click：单击触发
	  dblclick：双击触发
