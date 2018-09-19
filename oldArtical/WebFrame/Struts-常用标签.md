title: Struts-常用标签
date: 2016/3/9 15:58:24   
categories: WebFrame
---

# Struts-常用标签 #

Struts2常用标签总结

- Struts2的作用   
>- Struts2标签库提供了主题、模板支持，极大地简化了视图页面的编写，而且，struts2的主题、模板都提供了很好的扩展性。实现了更好的代码复用。
>- Struts2允许在页面中使用自定义组件，这完全能满足项目中页面显示复杂，多变的需求。
>- Struts2的标签库有一个巨大的改进之处，struts2标签库的标签不依赖于任何表现层技术，也就是说strtus2提供了大部分标签，可以在各种表现技术中使用。包括最常用的jsp页面，也可以说Velocity和FreeMarker等模板技术中的使用

- Struts2分类
> - （1）UI标签：（User  Interface, 用户界面）标签，主要用于生成HTML元素标签，UI标签又可分为表单标签非表单标签
> - （2）非UI标签，主要用于数据访问，逻辑控制等的标签。非UI标签可分为流程控制标签（包括用于实现分支、循环等流程控制的标签）和数据访问标签（主要包括用户输出ValueStack中的值，完成国际化等功能的
> - （3）ajax标签

- Struts2标签使用前的准备：
> 在要使用标签的jsp页面引入标签库： 

    <%@ taglib uri="/struts-tags" prefix="s"%>

4．标签的使用

（1）property标签

    用于输出指定的值：
    <s:property value="%{@cn.csdn.hr.domain.User@Name}"/><br/>
    <s:property value="@cn.csdn.hr.domain.User@Name"/><Br/><!-- 以上两种方法都可以 -->

    <s:property value="%{@cn.csdn.hr.domain.User@study()}"/> 以上可以访问某一个包的类的属性的集中方式，study()是访问方法的方法，并输出。

    以下用java代码代替的，访问某一个范围内的属性
    <%
    //采用pageContext对象往page范围内存入值来 验证#attr搜索顺序是从page开始的 ，搜索的顺序为：page，reques，session，application。
	set存值的时候存到的是request中，在jsp页面中访问的时候不用加任何的标识符，即可直接访问，如果不同的作用域不一样了，
			pageContext.setAttribute("name", "laoowang", PageContext.PAGE_SCOPE);   
	%>
	<s:property value="#attr.name" />

	假设在action中设置了不同作用域的类
		不同的作用域的标签的访问:   
	->获取的是requet中的对象值
		第一种方式:<s:property value="#request.user1.realName"/>
		<br/>
		第二种方式:<s:property value="#request.user1['realName']"/>
		<br/>
		第三种方式:<s:property value="#user1.realName"/>
	    <br/>
	    第四种方式:<s:property value="#user1['realName']"/>
	    <br/>
	    第五种方式：${requestScope.user1.realName }  || ${requestScope.user1['realName'] }
	    第六种方式：<s:property value="#attr.user1.realName"//attr对象按page==>  request sessionapplictio找的
	   
	->获取session中的值
	    第一种方式:<s:property value="#session.user1.realName"/>
	    第二种方式:<s:property value="#session.user1['realName']"/>
	 ..其它域类推

（2）iterator标签

	<s:iterator value="userList" status="st">
		<tr <s:if test="#st.odd">bgcolor="f8f8f8"</s:if> >
			<td align="center"><input type="checkbox" name="selectedRow" value="<s:property value='id'/>" /></td>
			<td align="center"><s:property value="name"/></td>
			<td align="center"><s:property value="account"/></td>
			<td align="center"><s:property value="dept"/></td>
			<td align="center"><s:property value="gender?'男':'女'"/></td>
			<td align="center"><s:property value="email"/></td>
			<td align="center">
			<a href="javascript:doEdit('<s:property value='id'/>')">编辑</a>
			<a href="javascript:doDelete('<s:property value='id'/>')">删除</a>
			</td>
		</tr>
	</s:iterator>
这个标签迭代的是一个list集合，集合中的每个元素是User对象，<s:property> 直接取值

（3）表单类标签

	<!-- 输入名字  对应input标签-->
    <td class="tdBg" width="200px">角色名称：</td>
    <s:textfield label="用户名" name="username" tooltip="你的名字" javascriptTooltip="false"></s:textfield>

    <s:textarea  name="rmake" cols="40" rows="20" tooltipDelay="300" tooltip="hi" label="备注" javascriptTooltip="true"></s:textarea>

     <s:password label="密码" name="password"></s:password>
     <s:file name="file1" label="上传文件"></s:file>
     <s:hidden name="id" value="1"></s:hidden>
             
    <!--
    <select name="edu">
        <option value="listKey">listValue</option>
     -->
    <s:select list="#{'1':'博士','2':'硕士'}" name="edu" label="学历" listKey="key" listValue="value"></s:select>
             
    <s:select list="{'java','.net'}" value="java"></s:select>   <!-- value是选中的 -->
             
    <!-- 必须有name -->
    <s:checkbox label="爱好 " fieldValue="true" name="checkboxFiled1"></s:checkbox>
             
    <!-- 多个checkbox -->
    <s:checkboxlist list="{'java','css','html','struts2'}" label="喜欢的编程语言" name="box" value="{'css','struts2'}"></s:checkboxlist>
         
         
     <!-- map集合前要加#   value作为显示值，key作为真值-->
    <s:checkboxlist list="#{1:'java',2:'css',3:'html',4:'struts2',5:'spring'}" label="喜欢的编程语言" name="boxs" value="{1,2}"></s:checkboxlist>
             
             
    <!-- listKey
                listValue
                 
                <input type="text" name="boxs" value="listKey">显示值listValue
     -->
                     
     <!-- radio -->       
       <%
                //从服务器传过来值
       ageContext.setAttribute("sex","男",PageContext.REQUEST_SCOPE);
       pageContext.setAttribute("sex1","男",PageContext.REQUEST_SCOPE);
      %>
     <s:radio list="{'男','女'}" name="sex" value="#request.sex"></s:radio>   
             
             
       <s:radio list="#{1:'男',2:'女'}" name="sex1" listKey="key" listValue="value" value="#request.sex1"></s:radio>        
         
       <!-- 防止表单提交的方式 -->
        <s:token></s:token>                    
        <s:submit value="提交"></s:submit>
