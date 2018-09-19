title: Struts2-文件上传与下载
date: 2016/3/9 15:58:07   
categories: WebFrame
---

# Struts2-文件上传与下载 #

> Struts2框架默认是支持文件上传的，并且从导入的jar包来看，实现的工具是fileupload工具。
> 文件上传的功能是由文件上传拦截器实现的。

## 文件上传 ##
在使用Struts2文件上传功能时，需注意：

### 上传表单的格式 ### 

	表单属性 enctype = multipart/form-data	   表单类型
	表单属性 method = post					   提交方式
	输入属性 type = file			               文件域

	//按格式定义，struts2会对表单进行自动封装到action的属性中
	<form action="${pageContext.request.contextPath }/fileUploadAction" method="post" enctype="multipart/form-data">
		用户名:<input type="text" name="userName"><br/>
		文件:<input type="file" name="file1"><br/>
		<input type="submit" value="上传">
	</form>

### Action的书写 ###
	public class FileUpload extends ActionSupport {   //
	
		// 对应表单：<input type="file" name="file1">
		private File file1;    //自动封装数据到这个文件对象
		// 文件名
		private String file1FileName;  //自动封装上传文件的名字
		// 文件的类型(MIME)
		private String file1ContentType;   //封装上传文件的类型
		
		public void setFile1(File file1) {
			this.file1 = file1;
		}
		public void setFile1FileName(String file1FileName) {
			this.file1FileName = file1FileName;
		}
		public void setFile1ContentType(String file1ContentType) {
			this.file1ContentType = file1ContentType;
		}
		
		
		@Override
		public String execute() throws Exception {
			/******拿到上传的文件，进行处理******/
			// 把文件上传到upload目录
			
			// 获取上传的目录路径
			String path = ServletActionContext.getServletContext().getRealPath("/upload");
			// 创建目标文件对象
			File destFile = new File(path,file1FileName);
			// 把上传的文件，拷贝到目标文件中
			FileUtils.copyFile(file1, destFile);   //将上传的文件，拷贝到本地。
	
			return SUCCESS;
		}
	}

### 文件上传的配置文件 ###
		<!-- 注意： action 的名称不能用关键字"fileUpload" ，可能会引起冲突-->
		<action name="fileUploadAction" class="cn.itcast.e_fileupload.FileUpload">
		
			<!-- 限制运行上传的文件的类型 -->
			<interceptor-ref name="defaultStack">			
				<!-- 限制运行的文件的扩展名 -->
				<param name="fileUpload.allowedExtensions">txt,jpg,jar</param>	
				<!-- 限制运行的类型   【与上面同时使用，取交集】
				<param name="fileUpload.allowedTypes">text/plain</param>
				-->	
			</interceptor-ref>
			
			<result name="success">/e/success.jsp</result>
			<!-- 配置错误视图 -->
			<result name="error">/e/error.jsp</result>
		</action>
		
配置常量：

	<!--  这里修改上传文件的最大大小为30M -->
	<constant name="struts.multipart.maxSize" value="31457280"/>

## 文件下载 ##

> Struts2的文件下载，核心是action返回的结果类型为"stream"。
> 我们需要详细配置返回结果`<result/>`

## action ##

	public class DownAction extends ActionSupport {
		
		/*************1. 显示所有要下载文件的列表*********************/
		public String list() throws Exception {
			
			//得到upload目录路径
			String path = ServletActionContext.getServletContext().getRealPath("/upload");
			// 目录对象
			File file  = new File(path);
			// 得到所有要下载的文件的文件名
			String[] fileNames =  file.list();
			// 保存
			ActionContext ac = ActionContext.getContext();
			// 得到代表request的map (第二种方式)
			Map<String,Object> request= (Map<String, Object>) ac.get("request");
			request.put("fileNames", fileNames);
			return "list";
		}
		
		/*************2. 文件下载*********************/
		
		// 1. 获取要下载的文件的文件名 ，————————》这里使用，请求数据的自动封装
		private String fileName;
		public void setFileName(String fileName) {
			// 处理传入的参数中问题(get提交)
			try {
				fileName = new String(fileName.getBytes("ISO8859-1"),"UTF-8");
			} catch (UnsupportedEncodingException e) {
				throw new RuntimeException(e);
			}
			// 把处理好的文件名，赋值
			this.fileName = fileName;
		}
		
		//2. 下载提交的业务方法 (在struts.xml中配置返回stream)
		public String down() throws Exception {
			return "download";
		}
		
		// 3. 返回文件流的方法
		public InputStream getAttrInputStream(){
			return ServletActionContext.getServletContext().getResourceAsStream("/upload/" + fileName);
		}
		
		// 4. 下载显示的文件名（浏览器显示的文件名）
		public String getDownFileName() {
			// 需要进行中文编码
			try {
				fileName = URLEncoder.encode(fileName, "UTF-8");
			} catch (UnsupportedEncodingException e) {
				throw new RuntimeException(e);
			}
			return fileName;
		}
	}

## 返回结果的配置 ##
	<action name="down_*" class="cn.itcast.e_fileupload.DownAction" method="{1}">
			<!-- 列表展示 -->
			<result name="list">/e/list.jsp</result>
			<!-- 下载操作 -->
			<result name="download" type="stream">
			
				<!-- 运行下载的文件的类型:指定为所有的二进制文件类型 -->
			   <param name="contentType">application/octet-stream</param>
			   
			   <!-- 对应的是Action中属性： 返回流的属性【其实就是getAttrInputStream()】 -->
			   <param name="inputName">attrInputStream</param>
			   
			   <!-- 下载头，包括：浏览器显示的文件名 -->
			   <param name="contentDisposition">attachment;filename=${downFileName}</param>	 
			 	<!-- 缓冲区大小设置 -->
			   <param name="bufferSize">1024</param>
			</result>
		</action>