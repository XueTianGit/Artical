title: Struts2-简单了解
date: 2016/3/9 15:57:34   
categories: WebFrame
---

#Struts2-数据回显、模型驱动等常用技术#

## 数据回显 ##

对于数据回显，必须要用struts标签！

例如下面这个标签有两种方式实现回显：

	<td><s:textfield name="user.username"/></td>

1） 将user作为action的实例变量
Action跳转时是这样的->

	public class UserAction extends ActionSupport {
		private User user;
		......
	
		public String editUI()
		{
			user = userService.findById(user.getId());
			return "editUI";
		}
	}

2）如果user不是实例变量，要想完成数据回显，应该这样写：

	public String editUI() {

		User user = new User();
		ActionContext ac = ActionContext.getContext();
		
		/************* 数据回显***************/
		// 获取值栈
		ValueStack vs = ac.getValueStack();
		vs.pop();// 移除栈顶元素
		vs.push(user);  // 入栈	
		// 进入修改页面
		return "editUI";
	}
并且这时的标签可以简写为：

    <s:textfield name="username"/>

综上所述，要想完成数据回显，必须将要回显的对象放在ValueStack的根元素上。这样Ognl表达式才能取到值，实现回显。

另外对于像下面这样的标签也能很方便的完成数据回显：

	<s:radio name="gender"  id="gender" list="#{'男':'男','女':'女'}"></s:radio>
	<s:select list="#session.departlist" listKey="id" listValue="dname" name="depart" ></s:select>     

## 模型驱动 ##

Struts运行时候，会执行默认的拦截器栈，其中有一个拦截器，模型驱动拦截器：

	<interceptor name="modelDriven" class="com.opensymphony.xwork2.interceptor.ModelDrivenInterceptor"/>
### 模型驱动与属性驱动的区别 ###

>属性驱动

对于属性驱动，我们需要在Action中定义与表单元素对应的所有的属性，因而在Action中会出现很多的getter和setter方法

>模型驱动

对于模型驱动，使用的Action对象需要实现ModelDriven接口并给定所需要的类型.而在Action中我们只需要定义一个封装所有数据信息的javabean

>属性和模型驱动的相同点

当我们使用属性驱动和模型驱动的时候，必须将表单的元素中的name属性值与我们定义接收数据信息的变量名对应起来。

很明显，模型驱动好像让我们写的东西更少了一些。。。。。。。

### 使用步骤 ###

	步骤：
	1. 实现ModelDriver接口
	2. 实现接口方法： 接口方法返回的就是要封装的对象
	3. 对象一定要实例化。

	/**
	 * 1. 数据回显
	 * 2. 模型驱动
	 *
	 */
	<input type=text name=userName />
  	<input type=text name=pwd />
	
	public class UserAction extends ActionSupport implements ModelDriven<User> {
		
		// 封装请求数据
		private User user = new User();
		public void setUser(User user) {
			this.user = user;
		}
		public User getUser() {
			return user;
		}
		
		// 实现模型驱动接口方法 , getModel方法自动将提交的数据给user属性赋值 
		@Override
		public User getModel() {
			return user;
		}
		
		
		public String add() {
			
			/*这里我们可以直接使用user的属性值*/
			return "success";
		}
		

## 防止表单重复提交 ##

> 可以使用<s:token />标签防止重复提交

	用法：
	第一步：在表单中加入<s:token />
		<s:form action="helloworld_other" method="post" namespace="/test">
	 	 	<s:textfield name="person.name"/><s:token/><s:submit/>
	  	</s:form>
	第二步：
		<action name="helloworld_*" class="cn.itcast.action.HelloWorldAction" method="{1}">
	         <interceptor-ref name="defaultStack"/>
	          <!-- 增加令牌拦截器 -->
	          <interceptor-ref name="token">
	                  <!-- 哪些方法被令牌拦截器拦截 -->
	                  <param name=“includeMethods">save</param>
	          </interceptor-ref>    
	         <!-- 当表单重复提交转向的页面 -->
	        <result name="invalid.token">/WEB-INF/page/message.jsp</result>  	
		</action>

以上配置加入了“token”拦截器和“invalid.token”结果，因为“token”拦截器在会话的token与请求的token不一致时，将会直接返回“invalid.token”结果。




