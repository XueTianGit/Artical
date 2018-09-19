title: Struts2-Results机制
date: 2016/3/9 15:56:03   
categories: WebFrame
---

# Struts2-Results机制 #

Struts2将Result列为一个独立的层次，可以说是整个Struts2的Action层架构设计中的另外一个精华所在。
Result之所以成为一个层次，其实是为了解决MVC框架中，如何从Control层转向View层这样一个问题而存在的。所以，接下来我们详细讨论一下Result的方方面面。 

## Result的职责  ##
Result作为一个独立的层次存在，必然有其存在的价值，它也必须完成它所在的层次的职责。Result是为了解决如何从Control层转向View层这样一个问题而存在的，
那么Result最大的职责，就是架起Action到View的桥梁。具体来说，我把这些职责大概分成以下几个方面： 

### 封装跳转逻辑  ###
Result的首要职责，是封装Action层到View层的跳转逻辑。之前我们已经反复提到，Struts2的Action是一个与Web容器无关的POJO。
所以，在Action执行完毕之后，框架需要把代码的执行权重新交还给Web容器，并转向到相应的页面或者其他类型的View层。 
而这个跳转逻辑，就由Result来完成。这样，好处也是显而易见的，对Action屏蔽任何Web容器的相关信息，使得每个层次更加清晰。 
View层的显示类型非常多，有最常见的JSP、当下非常流行的Freemarker/Velocity模板、Redirect到一个新的地址、文本流、图片流、甚至是JSON对象等等。所以Result层的独立存在，就能够对这些显示类型进行区分，并封装合理的跳转逻辑。 
以JSP转向为例，在Struts2自带的ServletDispatcherResult中就存在着核心的JSP跳转逻辑： 

    Java代码  收藏代码
    HttpServletRequest request = ServletActionContext.getRequest();  
    RequestDispatcher dispatcher = request.getRequestDispatcher(finalLocation);  
    ....  
      
    dispatcher.forward(request, response);  
    

再以Redirect重定向为例，在Struts2自带的ServletRedirectResult中，也同样存在着重定向的核心代码：

    Java代码  收藏代码
    HttpServletResponse response = (HttpServletResponse) ctx.get(ServletActionContext.HTTP_RESPONSE);  
      
    ....  
      
    response.sendRedirect(finalLocation);  


由此可见，绝大多数的Result，都封装了与Web容器相关的跳转逻辑，由于这些逻辑往往需要和Servlet对象打交道，
所以，遵循Struts2的基本原则，将它作为一个独立的层次，从而将Action从Web容器中解放出来。 

### 准备显示数据  ###

之前提到，View层的展现方式很多，除了传统的JSP以外，还有类似Freemarker/Velocity这样的模板。
根据模板显示的基本原理，需要将预先定义好的模板（Template）和需要展示的数据（Model）组织起来，交给模板引擎，
才能够正确显示。而这部分工作，就由Result层来完成。 
以Struts2自带的FreemarkerResult为例，在Result中，就存在着为模板准备数据的逻辑代码：
 

    Java代码  收藏代码
    protected TemplateModel createModel() throws TemplateModelException {  
	    ServletContext servletContext = ServletActionContext.getServletContext();  
	    HttpServletRequest request = ServletActionContext.getRequest();  
	    HttpServletResponse response = ServletActionContext.getResponse();  
	    ValueStack stack = ServletActionContext.getContext().getValueStack();  
      
	    Object action = null;  
	    if(invocation!= null ) action = invocation.getAction(); //Added for NullPointException  
	    return freemarkerManager.buildTemplateModel(stack, action, servletContext, request, response, wrapper);  
    }  


有兴趣的读者可以顺着思路去看源码，看看这些Result到底是如何获取各种对象的值的。 

### 控制输出行为  ###

有的时候，针对同一种类型的View展示，我们可能会有不同的输出行为。具体来说，可能有时候，
我们需要对输出流指定特定的BufferSize、Encoding等等。Result层，作为一个独立的层次，可以提供极大的扩展性，从而保证我们能够定义自己期望的输出类型。 
以Struts2自带的HttpHeaderResult为例： 

    Java代码  收藏代码
    public void execute(ActionInvocation invocation) throws Exception {  
	    HttpServletResponse response = ServletActionContext.getResponse();  
	      
	    if (status != -1) {  
	   	 response.setStatus(status);  
	    }  
	      
	    if (headers != null) {  
		    ValueStack stack = ActionContext.getContext().getValueStack();  
		      
		    for (Iterator iterator = headers.entrySet().iterator();  
		     iterator.hasNext();) {  
			    Map.Entry entry = (Map.Entry) iterator.next();  
			    String value = (String) entry.getValue();  
			    String finalValue = parse ? TextParseUtil.translateVariables(value, stack) : value;  
			    response.addHeader((String) entry.getKey(), finalValue);  
	  	   }  
	    } 
    }  


我们可以在这里添加我们自定义的内容到HttpHeader中去，从而控制Http的输出。 

## Result的定义  ##

让我们来看看Result的接口定义： 

	Java代码  收藏代码
	public interface Result extends Serializable {  
	  
	    /** 
	     * Represents a generic interface for all action execution results, whether that be displaying a webpage, generating 
	     * an email, sending a JMS message, etc. 
	     */  
	    public void execute(ActionInvocation invocation) throws Exception;  
	}  


这个接口定义非常简单，通过传入ActionInvocation，执行一段逻辑。
我们来看看Struts2针对这个接口实现的一个抽象类，它规定了许多默认实现： 

	Java代码  收藏代码
	public abstract class StrutsResultSupport implements Result, StrutsStatics {  
	  
	    private static final Log _log = LogFactory.getLog(StrutsResultSupport.class);  
	  
	    /** The default parameter */  
	    public static final String DEFAULT_PARAM = "location";  
	  
	    private boolean parse;  
	    private boolean encode;  
	    private String location;  
	    private String lastFinalLocation;  
	  
	    public StrutsResultSupport() {  
	        this(null, true, false);  
	    }  
	  
	    public StrutsResultSupport(String location) {  
	        this(location, true, false);  
	    }  
	  
	    public StrutsResultSupport(String location, boolean parse, boolean encode) {  
	        this.location = location;  
	        this.parse = parse;  
	        this.encode = encode;  
	    }  
	  
	    // setter method 省略  
	   
	    /** 
	     * Implementation of the <tt>execute</tt> method from the <tt>Result</tt> interface. This will call 
	     * the abstract method {@link #doExecute(String, ActionInvocation)} after optionally evaluating the 
	     * location as an OGNL evaluation. 
	     * 
	     * @param invocation the execution state of the action. 
	     * @throws Exception if an error occurs while executing the result. 
	     */  
	    public void execute(ActionInvocation invocation) throws Exception {  
	        lastFinalLocation = conditionalParse(location, invocation);  
	        doExecute(lastFinalLocation, invocation);  
	    }  
	  
	    /** 
	     * Parses the parameter for OGNL expressions against the valuestack 
	     * 
	     * @param param The parameter value 
	     * @param invocation The action invocation instance 
	     * @return The resulting string 
	     */  
	    protected String conditionalParse(String param, ActionInvocation invocation) {  
	        if (parse && param != null && invocation != null) {  
	            return TextParseUtil.translateVariables(param, invocation.getStack(),  
	                    new TextParseUtil.ParsedValueEvaluator() {  
	                        public Object evaluate(Object parsedValue) {  
	                            if (encode) {  
	                                if (parsedValue != null) {  
	                                    try {  
	                                        // use UTF-8 as this is the recommended encoding by W3C to  
	                                        // avoid incompatibilities.  
	                                        return URLEncoder.encode(parsedValue.toString(), "UTF-8");  
	                                    }  
	                                    catch(UnsupportedEncodingException e) {  
	                                        _log.warn("error while trying to encode ["+parsedValue+"]", e);  
	                                    }  
	                                }  
	                            }  
	                            return parsedValue;  
	                        }  
	            });  
	        } else {  
	            return param;  
	        }  
	    }  
	  
	    /** 
	     * Executes the result given a final location (jsp page, action, etc) and the action invocation 
	     * (the state in which the action was executed). Subclasses must implement this class to handle 
	     * custom logic for result handling. 
	     * 
	     * @param finalLocation the location (jsp page, action, etc) to go to. 
	     * @param invocation    the execution state of the action. 
	     * @throws Exception if an error occurs while executing the result. 
	     */  
	    protected abstract void doExecute(String finalLocation, ActionInvocation invocation) throws Exception;  
	}  


很显然，这个默认实现是为那些类似JSP，Freemarker或者Redirect这样的页面跳转的Result而准备的一个基类，
它规定了Result将要跳转到的具体页面的位置、是否需要解析参数，等等。 

如果我们试图编写自定义的Result，我们可以实现Result接口，并在struts.xml中进行声明： 

	public class CustomerResult implements Result {  
	  
	    public void execute(ActionInvocation invocation) throws Exception {  
	    // write your code here  
		}  
	}  

    <result-type name="customerResult" class="com.javaeye.struts2.CustomerResult"/>  

### 常用的Result ###

接下来，大致介绍一下Struts2内部已经实现的Result，并看看他们是如何工作的。 

#### dispatcher  ####

	<result-type name="dispatcher" class="org.apache.struts2.dispatcher.ServletDispatcherResult" default="true"/>  

dispatcher主要用于返回JSP，HTML等以页面为基础View视图，这个也是Struts2默认的Result类型。 
在使用dispatcher时，唯一需要指定的，是JSP或者HTML页面的位置，这个位置将被用于定位返回的页面：

	<result name="success">/index.jsp</result>  

而Struts2本身也没有对dispatcher做出什么特殊的处理，只是简单的使用Servlet API进行forward。
 
#### freemarker / velocity  ####


	<result-type name="freemarker" class="org.apache.struts2.views.freemarker.FreemarkerResult"/>  
	<result-type name="velocity" class="org.apache.struts2.dispatcher.VelocityResult"/>  

随着模板技术的越来越流行，使用Freemarker或者Velocity模板进行View层展示的开发者越来越多。
Struts2同样为模板作为Result做出了支持。由于模板的显示需要模板（Template）与数据（Model）的紧密配合，所以在Struts2中，这两个Result的主要工作是为模板准备数据。 

以Freemarker为例，我们来看看它是如何为模板准备数据的： 

	public void doExecute(String location, ActionInvocation invocation) throws IOException, TemplateException {  
	    this.location = location;  
	    this.invocation = invocation;  
	    this.configuration = getConfiguration();  
	    this.wrapper = getObjectWrapper();  
	  
	    // 获取模板的位置  
	    if (!location.startsWith("/")) {  
	        ActionContext ctx = invocation.getInvocationContext();  
	        HttpServletRequest req = (HttpServletRequest) ctx.get(ServletActionContext.HTTP_REQUEST);  
	        String base = ResourceUtil.getResourceBase(req);  
	        location = base + "/" + location;  
	    }  
	  
	    // 得到模板  
	    Template template = configuration.getTemplate(location, deduceLocale());  
	    // 为模板准备数据  
	    TemplateModel model = createModel();  
	  
	    // 根据模板和数据进行输出  
	    // Give subclasses a chance to hook into preprocessing  
	    if (preTemplateProcess(template, model)) {  
	        try {  
	            // Process the template  
	            template.process(model, getWriter());  
	        } finally {  
	            // Give subclasses a chance to hook into postprocessing  
	            postTemplateProcess(template, model);  
	        }  
	    }  
	}  


从源码中，我们可以看到，createModel()方法真正为模板准备需要显示的数据。而之前，我们已经看到过这个方法的源码，
这个方法所准备的数据不仅包含ValueStack中的数据，还包含了被封装过的HttpServletRequest，HttpSession等对象的数据。从而使得模板能够以它特定的语法输出这些数据。 
Velocity的Result也是类似，有兴趣的读者可以顺着思路继续深究源码。 

#### redirect  ####


    <result-type name="chain" class="com.opensymphony.xwork2.ActionChainResult"/>  
    <result-type name="redirect" class="org.apache.struts2.dispatcher.ServletRedirectResult"/>  
    <result-type name="redirectAction" class="org.apache.struts2.dispatcher.ServletActionRedirectResult"/>  


如果你在Action执行完毕后，希望执行另一个Action，有2种方式可供选择。一种是forward，另外一种是redirect。有关forward和redirect的区别，
这里我就不再展开，这应该属于Java程序员的基本知识。在Struts2中，分别对应这两种方式的Result，就是chain和redirect。 

先来谈谈redirect，既然是重定向，那么源地址与目标地址之间是2个不同的HttpServletRequest。
所以目标地址将无法通过ValueStack等Struts2的特性来获取源Action中的数据。如果你需要对目标地址传递参数，那么需要在目标地址url或者配置文件中指出： 


	<!--  
	The redirect-action url generated will be :  
	/genReport/generateReport.jsp?reportType=pie&width=100&height=100  
	-->  
	<action name="gatherReportInfo" class="...">  
	  <result name="showReportResult" type="redirect">  
	     <param name="location">generateReport.jsp</param>  
	     <param name="namespace">/genReport</param>  
	     <param name="reportType">pie</param>  
	     <param name="width">${width}</param>  
	     <param name="height">${height}</param>  
	  </result>  
	</action>  


同时，Redirect的Result支持在配置文件中，读取并解析源Action中ValueStack的值，并成为参数传递到Redirect的地址中。上面给出的例子中，width和height就是ValueStack中的值。 

#### stream  ####

	<result-type name="stream" class="org.apache.struts2.dispatcher.StreamResult"/>  


StreamResult等价于在Servlet中直接输出Stream流。这种Result被经常使用于输出图片、文档等二进制流到客户端。通过使用StreamResult，我们只需要在Action中准备好需要输出的InputStream即可。 

	<result name="success" type="stream">  
	  <param name="contentType">image/jpeg</param>  
	  <param name="inputName">imageStream</param>  
	  <param name="contentDisposition">filename="document.pdf"</param>  
	  <param name="bufferSize">1024</param>  
	</result>  


同时，StreamResult支持许多参数，对输出的Stream流进行参数控制。具体每个参数的作用，可以参考：http://struts.apache.org/2.0.14/docs/stream-result.html 

其他 

Struts2的高度可扩展性保证了许多自定义的Result可以通过插件的形式发布出来。比较著名的有JSONResult，JFreeChartResult等等。有兴趣的读者可以在Struts2的官方网站上找到它们，并选择合适的加入到你的项目中去。 
关于Result配置简化的思考  Top

Struts2的Result，解决了“如何从Control层转向View层”的问题。不过看了上面介绍的这些由框架本身实现的Result，我们可以发现Result所涉及到的，基本上还停留在为Control层到View层搭建桥梁。 

传统的，我们需要通过配置文件，来指定Action执行完毕之后，到底执行什么样的Result。不过在这样一个到处呼吁简化配置的年代，存在着许多方式，可以省略配置： 

- 使用Annotation 

Struts2的一些插件提供了@Result和@Results的Annotation，可以通过Annotation来省略XML配置。具体请参考相关的文档。 

- Codebehind插件 

Struts2自带了一个Codebehind插件（Struts2.1以后被合并到了其他的插件中）。Codebehind的基本思想是通过CoC的方式，使用命名约定来确定JSP等资源文件的位置。它通过实现了XWork的UnknownHandler接口，来实现当Struts2框架无法找到相应的Result时，如何进行处理的逻辑。具体文档可以参考： 

http://struts.apache.org/2.0.14/docs/codebehind-plugin.html 

大家可以在上面这两种方式中任意选择，国内著名的开源倡导者Springside也是采用了上述2种方法。在多数情况下，使用Codebehind，针对其他的一些Result使用Annotation进行配置，这样可以在一定程度上简化配置。 

不过我本人对使用Annotation简化配置的评价不高。因为实际上使用Annotation，只是将原本就非常简单的配置，从xml文件中移动到java代码中而已。就代码量而言，本身并没有减少。 

在这里，我也在经常在思考，如何进行配置简化，可以不写Annotation，完全使用CoC的方式来指定Result。Codebehind在CoC方面已经做出了榜样，只是Codebehind无法判别Result的类型，所以它只能支持dispatcher / freemarker / velocity这三种Result。所以Result的类型的判别，成为了阻碍简化其配置CoC化的拦路虎。 

前一段时间，曾经热播一部电视剧《暗算》，其中的《看风》篇中数学家黄依依的一段话给了我灵感： 

黄依依 写道
开启密锁钥匙的复杂化，是现代密码发展的趋势。但这种复杂化却受到无线通讯本身的限制，尤其是距离远、布点多的呈放射性的无线通讯，一般的密钥总是要藏在报文中。


密钥既然可以藏在报文中，那么Result的类型当然也能够藏在ResultCode中。 


这样一个简单的success作为ResultCode，是无法识别成复杂的Result类型的，我们需要设计一套更加有效的ResultCode，同时，Struts2能够识别这些ResultCode，并得到相应的Result类型和Result实例。这样，我们就可以借用Codebehind的实现方式，实现XWork的UnknownHandler接口，从而达到我们的目的。例如，我们规定ResultCode的解析规则： 

	success —— 使用codebehind的规则进行JSP，Freemarker模板的寻址 
	
	r:/user/list  —— 返回一个redirect的Result，地址为/user/list 
	
	c:/user/list —— 返回一个chain的Result，地址为/user/list 
	
	j:user —— 返回一个JSON的Result，JSONResult的Root对象为user 
	
	s:inputStream-text/html —— 返回一个StreamResult，使用inputStream，并将contentType设置成text/html 
