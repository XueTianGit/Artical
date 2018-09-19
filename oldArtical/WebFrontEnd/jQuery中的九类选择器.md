title: jQuery中的九类选择器
date: 2016/2/29 19:17:35                
categories: WebFrontEnd
---

# jQuery中的九类选择器 #

> 目的：通过九类选择器，能定位web页面（HTML/JSP/XML）中的任何标签

	1. 基本选择器
	-> #id: 通过元素的 id 属性中给定的值
		element:根据给定的元素名匹配所有元素
		.class  根据给定的类匹配元素。    $(".myClass");
		* 用于匹配所有元素
		selector1,selector2,selectorN   将每一个选择器匹配到的元素合并后一起返回
	2. 基本增强型  （当返回一堆结果时）
		:first  获取第一个元素    $('li').first()  or  $("li:first")
		:last  同上
		:not  去除所有与给定选择器匹配的元素  $("input:not(:checked)")  
		:even   匹配所有索引值为偶数的元素，从 0 开始计数    $("tr:even")
		:odd    。。。。。。。。奇数
		:eq     匹配一个给定索引值的元素    $("tr:eq(1)")
		:gt     匹配所有大于给定索引值的元素   $("tr:gt(0)")
		:lt             小于
		:header 匹配如 h1, h2, h3之类的标题元素
		:animated  匹配所有正在执行动画效果的元素
	3. 层级
	    ancestor descendant   在给定的祖先元素下匹配**所有的后代元素**    $("form input")
		parent > child        在给定的父元素下匹配所有的子元素-->**仅儿子， 不包括孙子什么的	**
	  	prev + next           匹配所有紧接在 prev 元素后的 next 元素    （两人连在一块了）
	    prev ~ siblings       匹配 prev 元素之后的所有 siblings 元素   （找他后面的兄弟）   $("form ~ input")
	4. 内容 
	    :contains  			  匹配包含给定文本的元素   $("div:contains('John')")  -> 查找内容包含John的div
		:empty				  匹配所有不包含子元素或者文本的空元素    -> 光棍
		:has				  匹配含有选择器所匹配的元素的元素   ->(找指定的儿子元素)            $("div:has(p)").addClass("test");
	    :parent  			  匹配含有子元素或者文本的元素      例如 ： 查找所有含有子元素或者文本的 td 元素     $("td:parent")
	5. 可见性
		：hiden 				  匹配所有不可见(display)元素，或者type为hidden的元素	  ->  找样式为display和hidden
		：visible
	6. 属性
		[attribute]			  匹配包含给定属性的元素。   $("div[id]")
		[attribute=value]     属性为特定值
		[attribute！=value]
		[attribute^=value]    匹配给定的属性是以某些值开始的元素   ->  找的是属性的内容（头）了
		[attribute$=value]	  匹配给定的属性是以某些值结尾的元素   ->  尾
		[attribute*=value]    匹配给定的属性是以包含某些值的元素   ->  包含
		[selector1][selector2][selectorN]      复合属性选择器，需要同时满足多个条件时使用  
	7. 子元素
		:nth-child            匹配其父元素下的第N个子或奇偶元素     :nth-child(2)   :nth-child(odd)
	    :first-child			  匹配第一个子元素  
		:last-child
		:only-child           如果某个元素是父元素中唯一的子元素，那将会被匹配   如果父元素中含有其他元素，那将不会被匹配。  ->  独生子
	8. 表单
		:input               匹配所有 input, textarea, select 和 button 元素
		:text  匹配所有文本框  类似的有 :paddword  :image :button :submit  : radio :checkbox :reset :file :hidden 
	9. 表单对象属性
		：enable  匹配所有可用元素
		 类似  ->  :disable  
		:select   匹配所有选中的option元素
		:checked 匹配所有选中的被选中元素(复选框、单选框等，不包括select中的option)




   