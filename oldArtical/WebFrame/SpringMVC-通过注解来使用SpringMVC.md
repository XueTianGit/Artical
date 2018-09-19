title:  SpringMVC-通过注解来使用SpringMVC
date: 2016/3/9 9:19:20  
categories: WebFrame
---

# SpringMVC-通过注解来使用SpringMVC #

> 前面我们使用过基于XML方式，来使用SpringMVC，感觉受到了很多的限制，而且很不好用，
> 基于注解来使用SpringMVC可以很好的满足我们的需要

> Spring MVC 中，如果我们没有注册任何 适配器、映射器、视图解析器到Spring容器中。我们依然可以使用注解，
> 因为 DispatcherServlet 将启用后备的几个默认 适配器、映射器、视图解析器实现 来支持注解。

	org.springframework.web.servlet.mvc.AnnotationMethodHandlerAdaptor 
	org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping
	org.springframework.web.servlet.view.InternalResourceViewResolver

所以说我们可以直接使用注解方式来配置SpringMVC。不过必须开启注解扫描。即：	<`context:component-scan base-package="xxxx"/>`


常用到的注解有@Controller和@RequestMapping

> - @Controller: 用于指明一个一个类是Controller
> - @RequestMapping: 可以用于映射请求路径， 指明业务方法处理的请求类型
> - @InitBinder：可用于自定义类型转换器


	下面这段代码：
	1）定义了一个Action （使用@Controller）
	2）映射了Action的访问路径和业务方法的访问路径，并规定方法处理什么样的请求（@RequestMapping）
	3）自定义了将Sting格式的日期转换为Date格式（@InitBinder）
	4）直接返回了字符串（真实路径）。（SpringMVC会自动根据路径去寻找相应页面）
	
	@Controller
	@RequestMapping(value="/user")
	public class UserAction {
		/**
		 * 自定义类型转换器
		 */
		@InitBinder
		public void initBinder(HttpServletRequest request,ServletRequestDataBinder binder) throws Exception {
			binder.registerCustomEditor(
					Date.class, 
					new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"),true));
		}
		/**
		 * 用户注册，只能接收POST请求
		 */
		@RequestMapping(method=RequestMethod.POST,value="/register")
		public String registerMethod(User user,Model model) throws Exception{
			System.out.println("用户注册:" + user.toString());
			//将user绑定到model对象中
			model.addAttribute("user",user);
			//转发到success.jsp页面
			return "/jsp/success.jsp";
		}
	}

注意点：

- 1）value可以接受一个数组， 即一个业务方法可以配置多个访问路径
- 2）如果没用指定@RequestMapping的method属性， 那么默认方法是可以处理任何方式的请求
- 3）使用Model对象， 把返回数据进行封装。




## 对于请求参数的接收##
> 从上面的业务方法可以看出， 我们可以这样接收请求参数
> 
> - 直接在业务方法的参数上写上相应的名字， 那么SpringMVC会自动封装到这个参数中。


> - 当参数较多时（例如表格）， 可以将参数封装为实体（例如User）。 SpringMVC还是可以自动封装的  （就像Struts2， 不过更强大）
	但是，使用实体封装请求参数时，应注意：若前台传过来许多参数，而我们使用两个实体封装的话。
	这时，为了避免混乱，我们不应再直接在业务方法上简单写上这两个实体参数来接收了， 可以将这两个参数再封装，
	但是此时前台在提交请求参数时，就应该加上对象名了，例如：


	/*前台携带的请求数据*/
	<td><input type="text" name="user.username" value="${user.username}"/></td>   
	...
	<td><input type="text" name="admin.username" value="${admin.username}"/></td>
	....

	/*这个Bean封装两个实体：User、Admin*/
	public class Bean {
		private User user;
		private Admin admin;
		/*...*/
	}
	
	/*处理前台请求的Action*/
	@Controller
	@RequestMapping(value="/person")
	public class PersonAction {
	/**
	 * 业务方法
	 */	
		@RequestMapping(value="/register")
		public String registerMethod(Bean bean,Model model) throws Exception{
			//将user和admin都绑定到Model
			model.addAttribute("user",bean.getUser());
			model.addAttribute("admin",bean.getAdmin());
			//转发
			return "/02_person.jsp";
		}
	}

### 以数组方式来收集前台数据 ###
当前台提交的数据多个名字都相同时， 我们可以使用数组来接收这些数据。

假如， 我们要批量删除数据， 前台传过来的是id，我们可以这样接收：

	/*前台数据  提交的 /deleAll.action*/
	<input type="checkbox" name="ids" value="1"/>
	<input type="checkbox" name="ids" value="2"/>
	<input type="checkbox" name="ids" value="3"/>
	.....
	
	/*Action处理*/
	public class EmpAction {
	
		@RequestMapping(value="/deleteAll",method=RequestMethod.POST)
		public String deleteAll(Model model,int[] ids) throws Exception{
			System.out.println("需要删除的员工编号分别是：");
			for(int id : ids){
				System.out.print(id+" ");
			}
			return "/jsp/ok.jsp";
		}
	}


如果要提交多个实体（例如批量增加Emp）， 我们可以这样来收集：

	/*前台*/
	<td><input type="text" name="empList[0].username" value="哈哈"/></td>
	<td><input type="text" name="empList[0].salary" value="7000"/></td>
	<td><input type="text" name="empList[1].username" value="呵呵"/></td>
	<td><input type="text" name="empList[1].salary" value="7500"/></td>

	/*封装实体的bean*/
	public class Bean {
		private List<Emp> empList = new ArrayList<Emp>();
		/*...*/
	}
	
	//业务方法
	@RequestMapping(value="/addAll",method=RequestMethod.POST)
	public String addAll(Model model,Bean bean) throws Exception{
		for(Emp emp:bean.getEmpList()){
			System.out.println(emp.getUsername()+":"+emp.getSalary());
		}
		model.addAttribute("message","批量增加员工成功");
		return "/jsp/ok.jsp";
	}


## 转发与重定向到Action ##

在Struts2中重定向到Action需要配置， 在SpringMVC中也可以，不过SpringMVC更加简洁。
只需：

	@Controller
	@RequestMapping(value="/user")
	public class UserAction {
		@RequestMapping(value="/delete")
		public String delete(int id) throws Exception{
				
			/*...*/
			return "forward:/user/find.action";  //转发到一个action
			//return "redirect:/user/find.action";			//重定向到action
		}


NT：对于转发而言，我们是可以共享请求参数的， 重定向肯定是不可以的啦。


## 返回JSON格式文本 ##
步骤：
> - 1）导入jackson-core-asl-1.9.11.jar和jackson-mapper-asl-1.9.11.jar
> - 2）配置适配器：AnnotationMethodHandlerAdapter
> - 3）在业务方法的返回值和权限之间使用@ResponseBody注解表示返回值对象需要转成JSON文本

范例代码：
	
	<!-配置适配器->
	<bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
			<property name="messageConverters">
					<list>
						<bean class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter"/>
					</list>
			</property>
	</bean>

	//Action
	@Controller
	@RequestMapping(value="/emp")
	public class EmpAction {	
		/**
	     * @ResponseBody Emp 表示让springmvc将Emp对象转成json文本
		 */
		@RequestMapping(value="/bean2json")
		public @ResponseBody Emp bean2json() throws Exception{
			//创建Emp对象
			Emp emp = new Emp();
			emp.setId(1);
			emp.setUsername("哈哈");
			emp.setSalary(7000D);
			emp.setHiredate(new Date());
			return emp;
		}
		
	}



