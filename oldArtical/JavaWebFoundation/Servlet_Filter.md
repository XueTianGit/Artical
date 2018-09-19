title: JavaWebFoundation- Servlet Filter
date: 2016/3/1 9:18:40               
categories: JavaWebFoundation
---


# Servlet Filter #

## 简介 ##
> - Filter也称之为过滤器，它是Servlet技术中最激动人心的技术，WEB开发人员通过Filter技术，
> 对web服务器管理的所有web资源
>    - 例如Jsp, Servlet, 静态图片文件或静态 html 文件等进行拦截，从而实现一些特殊的功能。
>    - 例如实现URL级别的权限访问控制、过滤敏感词汇、压缩响应信息等一些高级功能。


## Filter链 ##
- 在一个web应用中，可以开发编写多个Filter，这些Filter组合起来称之为一个Filter链。
web服务器根据Filter在web.xml文件中的注册顺序，决定先调用哪个Filter。
- 当第一个Filter的doFilter方法被调用时，
  - web服务器会创建一个代表Filter链的FilterChain对象传递给该方法。在doFilter方法中，开发    人员如果调用了FilterChain对象的doFilter方法，
   则web服务器会检查FilterChain对象中是否还有filter，如果有，则调用第2个filter，如果没有，则调用目标资源。

->  可以理解为环形绳， 众多Filter围着目标资源。

## Filter的生命周期 ##
- init(FilterConfig filterConfig)throws ServletException
> 和我们编写的Servlet程序一样，Filter的创建和销毁由WEB服务器负责。 web 应用程序启动时，web 服务器将创建Filter 的实例对象，并调用其init方法，完成对象的初始化功能，从而为后续的用户请求作好拦截的准备工作（注：filter对象只会创建一次，init方法也只会执行一次）
> 开发人员通过init方法的参数，可获得代表当前filter配置信息的FilterConfig对象。

- destroy()
> 在Web容器卸载 Filter 对象之前被调用。该方法在Filter的生命周期中仅执行一次。在这个方法中，可以释放过滤器使用的资源。

## FilterConfig接口 ##
- 用户在配置filter时，可以使用<init-param>为filter配置一些初始化参数，当web容器实例化Filter对象，调用其init方法时，会把封装了filter初始化参数的filterConfig对象传递进来。
- 因此开发人员在编写filter时，通过filterConfig对象的方法，就可获得：
>
	String getFilterName()：得到filter的名称。
	String getInitParameter(String name)： 返回在部署描述中指定名称的初始化参数的值。如果不存在返回null.
	Enumeration getInitParameterNames()：返回过滤器的所有初始化参数的名字的枚举集合。
	public ServletContext getServletContext()：返回Servlet上下文对象的引用。



## 开发使用步骤 ##
- 编写java类实现Filter接口，并实现其doFilter方法。
- 在 web.xml 文件中使用<filter>和<filter-mapping>元素对编写的filter类进行注册，并设置它所能拦截的资源。

### 映射Filter的细节 ###

	<filter-mapping>元素用于设置一个 Filter 所负责拦截的资源。一
	个Filter拦截的资源可通过两种方式来指定：Servlet 名称和资源访问的请求路径
	<filter-name>子元素用于设置filter的注册名称。该值必须是在<filter>元素中声明过的过滤器的名字
	<url-pattern>设置 filter 所拦截的请求路径(过滤器关联的URL样式)
	<servlet-name>指定过滤器所拦截的Servlet名称。
	<dispatcher>指定过滤器所拦截的资源被 Servlet 容器调用的方式，可以是REQUEST,INCLUDE,FORWARD和ERROR之一，默认REQUEST。用户可以设置多个<dispatcher> 子元素用来指定 Filter 对资源的多种调用方式进行拦截。

范例（解决乱码问题的过滤器）：
	public class FilterDemo1 implements Filter {
	
		private FilterConfig config;
		
		public void doFilter(ServletRequest request, ServletResponse response,
				FilterChain chain) throws IOException, ServletException {
			
			String value = this.config.getInitParameter("xxx");
			
			request.setCharacterEncoding("UTF-8");
			response.setCharacterEncoding("UTF-8");
			response.setContentType("text/html;charset=UTF-8");
			
			chain.doFilter(request, response);  //放行
	
		}
	
		public void init(FilterConfig filterConfig) throws ServletException {
			this.config = filterConfig;
		}
		
		public void destroy() {
		}
	}
	
		<filter>
			<filter-name>FilterDemo1</filter-name>
			<filter-class>cn.suixin.web.filter.FilterDemo1</filter-class>
			<init-param>
				<param-name>xxx</param-name>
				<param-value>yyyy</param-value>
			</init-param>
		</filter>
		<filter-mapping>
			<filter-name>FilterDemo1</filter-name>
			<url-pattern>/*</url-pattern>
		</filter-mapping>
		
## Filter高级开发 ##
> 由于开发人员在filter中可以得到代表用户请求和响应的request、response对象，因此在编程中可以使用Decorator(装饰器)模式对request、response对象进行包装，
> 再把包装对象传给目标资源，从而实现一些特殊需求。

### 已经实现的包装类 ###
- Servlet API 中提供了一个request对象的Decorator设计模式的默认实现类，
 	- 	HttpServletRequestWrapper 
 	-  （HttpServletRequestWrapper 类实现了request 接口中的所有方法，
 	    但这些方法的内部实现都是仅仅调用了一下所包装的的 request 对象的对应方法）
    - 避免用户在对request对象进行增强时需要实现request接口中的所有方法。
- 类似的还有ttpServletRequestWrapper 



## Fileter常见应用 ##
1. 统一全站字符编码的过滤器   (已进行过总结)
2. 控制浏览器缓存页面中的静态资源的过滤器

代码核心：  我们缓存住这些静态资源， 因此要对浏览器的输出进行拦截，然我缓存住， 再输出
			用到：  过滤器 装饰设计模式， ByteArrayOutputStream（缓存流）
	public class WebCacheFilter implements Filter {
		
		private Map<String,byte[]> map = new HashMap();
		
		public void doFilter(ServletRequest req, ServletResponse resp,
				FilterChain chain) throws IOException, ServletException {
			
			HttpServletRequest request = (HttpServletRequest) req;
			HttpServletResponse response = (HttpServletResponse) resp;
			
			//1.得到用户想访问的资源（uri）
			String uri = request.getRequestURI();
			
			//2.看map集合中是否保存了该资源的数据
			byte b[] = map.get(uri);
			
			//3.如果保存了，则直接取数据打给浏览器
			if(b!=null){
				response.getOutputStream().write(b);
				return;
			}
			
			//4.如果没有保存数据，则放行让目标资源执行，这时还需写一个response的包装类，捕获目标资源的输出
			MyResponse my = new MyResponse(response);
			chain.doFilter(request, my);
			byte data[] = my.getBuffer();
			
			//5.以资源uri为关键字，打资源的数据保存map集合中，以备于下次访问
			map.put(uri, data);
			
			//6.输出数据给浏览器
			response.getOutputStream().write(data);
			
		}
		
		class MyResponse extends HttpServletResponseWrapper{
			private ByteArrayOutputStream bout = new ByteArrayOutputStream();
			private PrintWriter pw;
			
			private HttpServletResponse response;
			public MyResponse(HttpServletResponse response) {
				super(response);
				this.response = response;
			}
			@Override
			public ServletOutputStream getOutputStream() throws IOException {
				return new MyServletOutputStream(bout);    //myresponse.getOutputStream().write("hahah");
			}
			
			@Override
			public PrintWriter getWriter() throws IOException {
				pw = new PrintWriter(new OutputStreamWriter(bout,response.getCharacterEncoding()));
				return pw;  //MyResponse.getWriter().write("中国");
			}
			public byte[] getBuffer(){
				if(pw!=null){
					pw.close();
				}
				return bout.toByteArray();
			}
		}
		
		class MyServletOutputStream extends ServletOutputStream{
	
			private ByteArrayOutputStream bout;
			public MyServletOutputStream(ByteArrayOutputStream bout){
				this.bout = bout;
			}
			@Override
			public void write(int b) throws IOException {
				bout.write(b);
			}
			
		}
	
		public void init(FilterConfig filterConfig) throws ServletException {
			// TODO Auto-generated method stub
	
		}
		
		public void destroy() {
			// TODO Auto-generated method stub
	
		}
	
	}


## 禁止浏览器缓存所有动态页面的过滤器 ##

核心： 有 3 个 HTTP 响应头字段都可以禁止浏览器缓存当前页面，它们在 Servlet 中的示例代码如下：

	response.setDateHeader("Expires",-1);
	response.setHeader("Cache-Control","no-cache"); 
	response.setHeader("Pragma","no-cache"); 

我们只需在过滤器的代码中，把response的这3个头设置了就OK了。

> NT： 这样设是因为->
> 并不是所有的浏览器都能完全支持上面的三个响应头，因此最好是同时使用上面的三个响应头。

- Expires数据头：值为GMT时间值，为-1指浏览器不要缓存页面
- Cache-Control响应头有两个常用值： 
- no-cache指浏览器不要缓存当前页面。
- max-age:xxx指浏览器缓存页面xxx秒。

## 使用Filter实现URL级别的权限认证 ##
看谁不顺眼，就让他滚蛋
核心： String uri = request.getRequestURI();