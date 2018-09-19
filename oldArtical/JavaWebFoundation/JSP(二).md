title: JavaWebFoundation-JSP技术（二）
date: 2016/3/1 9:18:40               
categories: JavaWebFoundation
---

# JSP技术（二） #

## JSP标签 ##
> 它用于在JSP页面中提供业务逻辑功能，避免在JSP页面中直接编写Java代码（可以自定义标签，来代替Java代码）。

常用JSP标签：

	<jsp:include page=""/>:动态包含,a包含b,a在执行过程中调用b,page的值可以是变量.它总是会检查所含文件中的变化，适合用于包含动态页面，并且可以带参数 
 	<%@ include file=""%>:静态包含,a包含b,**编译过程中a会受到b的jsp指令的影响**,例如,b中有一个isELIgnored指令,会影响b,效率更高,不仅仅是引入输出内容,引入共同的的编译元素,不会检查所含文件的变化，适用于包含静态页面 
	<jsp:forward>:此forward调用的pageContext的forward然后return了
	<jsp:param>:用于给上面两个标签传参数.

## JSP映射 ##
> JSP最终会被翻译成Servlet， 因此JSP的映射类似Servlet映射

	example：
	    <servlet>
	    	<display-name>Demo</display-name>
	    	<jsp-file>index.jsp</jsp-file>
	    </servlet>
	    <servlet-mapping>
	    	<servlet-name>Demo</servlet-name>
	    	<url-pattern>/index1.html</url-pattern>
	    </servlet-mapping>

### JSP开发模式 ###
- JSP+JavaBean
> 在模式一中，JSP页面独自响应请求并将处理结果返回客户。所有的数据通过Bean来处理JSP实现页面的表现。模式一技术也实现了页面的表现--和页面的商业逻辑相分离。大量使用模式形式，常常会导致页面被嵌入大量的脚本语言或JAVA代码。当需要处理的商业逻辑很复杂时，这种情况会变得非常糟糕。大量的代码会使整个页面变得非常复杂。对于前端界面设计人员来说，这简直不可想象,维护比较困难。
> 它不能满足大型项目的需要，但是可以较好的满足小型应用，在简单的应用中可以考虑模式一。

范例： 开发一个简单的计算器
	<jsp:useBean id="caculator" class="com.suixin.bean.CaculatorBean"></jsp:useBean>   //创建用于封装数据的bean
	<jsp:setProperty property="*" name="caculator"/>                                   //将表单提交的数据设置到bean中
	
	//处理数据
	<%
		try
		{
			caculator.caculate();			
		}
		catch(Exception e)
		{
			out.write(e.getMessage());
		}

	%>
	
	<br/>-----------------------------------------------------------------------------------------<br/>
	//获取数据
	The result of caculation:
	<jsp:getProperty property="firstNum" name="caculator"/>
	<jsp:getProperty property="operator" name="caculator"/>	
	<jsp:getProperty property="secondNum" name="caculator"/>
	=
	<jsp:getProperty property="result" name="caculator"/>
	
	<br/>-----------------------------------------------------------------------------------------<br/>
	
	//表单将数据提交，保存到bean中
	<form  action="/JSP/caculator.jsp" method="post">
		<table width="40%" border="1" >
			<tr>
				<td colspan="2">Caculator</td>
			</tr>
			<tr>
				<td>Parameter 1</td>
				<td>
					<input type="text" name="firstNum">
				</td>
			</tr>
			
			<tr>
				<td>Operator</td>
				<td>
					<select name="operator">
					<option value="+">+</option>
					<option value="-">-</option>
					<option value="*">*</option>
					<option value="/">/</option>
					</select>
				</td>
			</tr>
				<td>Parameter 2</td>
				<td>
					<input type="text" name="secondNum">
				</td>			
			<tr>
				<td colspan="2">
					<input type="submit" value="submit">
				</td>
			</tr>
		</table>
	</form>

- JSP+JavaBean+Servlet
> 在这个模式中 JSP来表现页面。通过Servlet来完成大量的事务处理，javabean则用来封装数据。Servlet充当一个控制者的角色，并负责向客户发送请求。Servlet创建JSP所需要的Bean和对象，然后根据用户的请求行为，决定将哪个JSP页面发送给客户。（MVC）

> 从开发的观点，模式二具有更清晰的页面表现，清楚的开发者角色划分，可以充分利用开发小组的界面设计人员，这些优势在大型项目开发中表现的尤为突出，使用这一模式，可以充分发挥每个开发者各自的特长，界面设计人员可以充分发挥自己的表现力，设计出优美的界面表现形式，设计人员可以充分发挥自己的商务处理思维，来实现项目中的业务处理。在大型项目中，模式二更被采用。