title: JQuery-EasyUI组件
date: 2016/2/29 19:16:09            
categories: WebFrontEnd
---

# JQuery-EasyUI组件 #
官方介绍：
JQueryEasyUI框架帮助你更容易的创建web页面
> - EasyUI是基于jQuery的一套用户界面组件集合
> - EasyUI提供了创建现代化，交互性的必要的JS应用程序
> - 使用EasyUi后， 你不必再要太多的JS代码，只需要一些关键的HTML标签中定义标识符
> - 对于HTML5这样的Web页面，EasyUI也能完成任务
> - EasyUI是一种非常强大且简单的技术

NT：当EasyUI版本过高时，则需要高版本的浏览器支持。


> EasyUI用在什么地方呢？
> 它可以快速构建比较漂亮的web页面，**一般用在构建web后台页面**。

回顾一下学过的前端技术都用在什么地方：
> - JS：基于浏览器对web页面中的节点进行操作，比较麻烦
> - jQuery：基于浏览器简化对web页面中的节点进行操作，做到了write less do more
> - AJAX：基于浏览器与服务端进行局部刷新的异步通讯编程模式
> - JSON：简化自定义对象的创建与AJAX数据交换轻量级文本
> - EasyUI：快速基于现成的组件创建自已的web页面

**EasyUI的学习非常简单， 因为官方提供了非常清晰的开发文档。**
这里只做简单介绍。（其实我也只是简单使用了一下。。。。）

## 导入easyui库 ##
> 这里以EasyUI的layout布局组件和可折叠功能的面板组件，来简单了解一下easyUi

->当然， 首先我们应导入官方库：

	在web工程中的WebRoot目录下创建js和themes目录，导入官方文件
	在要使用的html、jsp中导入css和js文件：

  	<!-- 引入外部CSS文件 -->
    <link rel="stylesheet" href="../themes/icon.css" type="text/css"></link>
    <link rel="stylesheet" href="../themes/default/easyui.css" type="text/css"></link>
  
  	<!-- 引入外部JS文件 -->
  	<script type="text/javascript" src="../js/jquery.min.js"></script>   // 这两个必须要有顺序
  	<script type="text/javascript" src="../js/jquery.easyui.min.js"></script>


## 如何使用 ##
> - 这里以EasyUI窗口组件为例，演示EasyUI组件的使用。
> - 组件的加载方式分为两种：静态加载和动态加载。
> - 一个组件一般都包括一组属性、事件和方法。


### 静态加载 ###

	创建窗口：
	<div id="win" class="easyui-window" title="My Window" style="width:600px;height:400px"   
	        data-options="iconCls:'icon-save',modal:true">   
	    Window Content    
	</div>  

静态加载关键点：
> - 1）class属性，应写对应的组件名
> - 2）只有class属性存在， 我们才可以写data-options属性
> - 3）data-options是设置组件的各种属性的。属性可以去文档中查阅。


### 动态加载 ###

	1.定义一个容器
	   <div id="win"></div>  
	2.通过JS创建窗口
		$('#win').window({    
		    width:600,    
		    height:400,    
		    modal:true   
		});  

动态加载关键点：
> - 1）得有容器。 具体用什么样的标签做容器， 去文档查阅
> - 2）动态固定加载格式
> 	$("容器标签").plugin({json对象})
> 
> 	plugin：为组件的名称
> 	json对象：规定组件的各种属性











