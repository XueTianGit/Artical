title: Struts-自定义的MyStruts框架
date: 2016/3/9 15:58:32   
categories: WebFrame
---

# Struts-自定义的MyStruts框架 #
 
> 前面在总结SSH时，对于Struts当时说到他非常依赖于Servlet，当时有点不明白， 于是又把当初看传智博客视频时老师写的MyStruts代码看了一下，
> 发现，哦，原来是这么回事。

代码：

## mystruts.xml 

	<?xml version="1.0" encoding="UTF-8"?>
	<mystruts>
		<package>
			<action name="login" class="cn.itcast.framework.action.LoginAction" method="login">
				<result name="loginSuccess" type="redirect">/index.jsp</result>
				<result name="loginFaild">/login.jsp</result>
			</action>		
			<action name="register" class="cn.itcast.framework.action.RegisterAction" method="register">
				<result name="registerSuccess">/login</result>
			</action>		
		</package>
	</mystruts>

## 3个用来管理Action的类 ##

ActionMapping类：（一个bean get、set方法省略）
	
	public class ActionMapping {
	
		// 请求路径名称
		private String name;
		// 处理aciton类的全名
		private String className;
		// 处理方法
		private String method;
		// 结果视图集合
		private Map<String, Result> results;
		/* 下面省略了get、set方法*/
	}

Result： 封装结果集

	public class Result {
	
		// 跳转的结果标记
		private String name;
		// 跳转类型，默认为转发； "redirect"为重定向
		private String type;
		// 跳转的页面
		private String page;	
		/* 下面省略了get、set方法*/
	}


ActionMappingManager： 加载配置文件, 封装所有的整个mystruts.xml

	public class ActionMappingManager {

	// 保存action的集合
	private Map<String,ActionMapping> allActions ;
	
	public ActionMappingManager(){
		allActions = new HashMap<String,ActionMapping>();
		// 初始化
		this.init();
	}
	
	/**
	 * 根据请求路径名称，   返回Action的映射对象
	 * @param actionName   当前请求路径
	 * @return             返回配置文件中代表action节点的AcitonMapping对象
	 */
	public ActionMapping getActionMapping(String actionName) {
		if (actionName == null) {
			throw new RuntimeException("传入参数有误，请查看struts.xml配置的路径。");
		}
		
		ActionMapping actionMapping = allActions.get(actionName);
		if (actionMapping == null) {
			throw new RuntimeException("路径在struts.xml中找不到，请检查");
		}
		return actionMapping;
	}
	
	// 初始化allActions集合
	private void init() {
		/********DOM4J读取配置文件***********/
		try {
			// 1. 得到解析器
			SAXReader reader = new SAXReader();
			// 得到src/mystruts.xml  文件流
			InputStream inStream = this.getClass().getResourceAsStream("/mystruts.xml");
			// 2. 加载文件
			Document doc = reader.read(inStream);
			
			// 3. 获取根
			Element root = doc.getRootElement();
			
			// 4. 得到package节点
			Element ele_package = root.element("package");
			
			// 5. 得到package节点下，  所有的action子节点
			List<Element> listAction = ele_package.elements("action");
			
			// 6.遍历 ，封装
			for (Element ele_action : listAction) {
				// 6.1 封装一个ActionMapping对象
				ActionMapping actionMapping = new ActionMapping();
				actionMapping.setName(ele_action.attributeValue("name"));
				actionMapping.setClassName(ele_action.attributeValue("class"));
				actionMapping.setMethod(ele_action.attributeValue("method"));
				
				// 6.2 封装当前aciton节点下所有的结果视图
				Map<String,Result> results = new HashMap<String, Result>();
				
				// 得到当前action节点下所有的result子节点
				 Iterator<Element> it = ele_action.elementIterator("result");
				 while (it.hasNext()) {
					 // 当前迭代的每一个元素都是 <result...>
					 Element ele_result = it.next();
					 
					 // 封装对象
					 Result res = new Result();
					 res.setName(ele_result.attributeValue("name"));
					 res.setType(ele_result.attributeValue("type"));
					 res.setPage(ele_result.getTextTrim());
					 
					 // 添加到集合
					 results.put(res.getName(), res);
				 }
				
				// 设置到actionMapping中
				actionMapping.setResults(results);
				
				// 6.x actionMapping添加到map集合
				allActions.put(actionMapping.getName(), actionMapping);
			}	
		} catch (Exception e) {
			throw new RuntimeException("启动时候初始化错误",e);
		}
	}
	}

## 核心Servlet ##

	public class ActionServlet extends HttpServlet{
		
		private ActionMappingManager actionMappingManager;
		
		// 只执行一次  (希望启动时候执行)
		@Override
		public void init() throws ServletException {
			actionMappingManager = new ActionMappingManager();
		}
	
		// http://localhost:8080/mystruts/login.action
		@Override
		protected void doGet(HttpServletRequest request, HttpServletResponse response)
				throws ServletException, IOException {
			
			try {
				// 1. 获取请求uri, 得到请求路径名称   【login】
				String uri = request.getRequestURI();
				// 得到 login
				String actionName=uri.substring(uri.lastIndexOf("/")+1, uri.indexOf(".action"));
				
				// 2. 根据路径名称，读取配置文件，得到类的全名   【cn..action.LoginAction】
				ActionMapping actionMapping = actionMappingManager.getActionMapping(actionName);
				String className = actionMapping.getClassName();
				
				// 当前请求的处理方法   【method="login"】
				String method = actionMapping.getMethod();
				
				// 3. 反射： 创建对象，调用方法； 获取方法返回的标记
				Class<?> clazz = Class.forName(className);
				Object obj = clazz.newInstance();  //LoginAction loginAction = new LoginAction();
				Method m = clazz.getDeclaredMethod(method, HttpServletRequest.class,HttpServletResponse.class );
				// 调用方法返回的标记
				String returnFlag =  (String) m.invoke(obj, request, response);
				
				// 4. 拿到标记，读取配置文件得到标记对应的页面 、 跳转类型
				Result result = actionMapping.getResults().get(returnFlag);
				// 类型
				String type = result.getType();
				// 页面
				String page = result.getPage();
				
				// 跳转
				if ("redirect".equals(type)) {
					response.sendRedirect(request.getContextPath() + page);
				} else {
					request.getRequestDispatcher(page).forward(request, response);
				}
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
		@Override
		protected void doPost(HttpServletRequest req, HttpServletResponse resp)
				throws ServletException, IOException {
			doGet(req, resp);
		}
	}