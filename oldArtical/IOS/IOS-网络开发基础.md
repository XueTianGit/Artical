title: IOS-网络开发基础
date: 2016/3/9 8:51:22               
categories: IOS
---

# IOS-网络开发基础 #

>先看个控件热热身

## UIWebView##
>- 类似与Android中的WebView，使用他可以很轻易的实现一个浏览器
>- 但是，UIWebView有很多强大之处
>    - 能够加载html/htm、pdf、docx、txt等格式的文件
>    - 系统自带的Safari浏览器就是通过UIWebView实现的
>    - 通过设置 webView的dataDetectorTypes = UIDataDetectorTypeAll;
>      可使webView对于其显示的内容进行检查并标记高亮(比如电话，邮箱等)
>- UIWebView还可以加载Bundle或者沙盒中的文件

- 基本使用
>- 即如何利用UiWebView来加载一个URL
	
	// 1. 确定要访问的资源——URL
	NSURL *url = [NSURL URLWithString:@"http://www.baidu.com"];
    // 2. 在URL中，如果包含中文字符串，需要将字符串转换为带百分号的格式(Base64编码？)
    url = [NSURL URLWithString:[url stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding]];
	
	// 2. 建立网络请求
	NSURLRequest *request = [NSURLRequest requestWithURL:url];

	// 3. UIWebView加载网络请求
	[self.webView loadRequest:request];


- UIWebView的代理及其属性
>- UIWebView的代理方法较少，都是用来监听资源加载进程的
>- 有两个布尔变量可以用来查看浏览器是否可以向前查看内容或者向后查看内容
>- 对于浏览器的前进和后退功能的实现，可以直接通过在storyboard中通过将(Received Actions)对按钮的拖线来实现

- 加载加载Bundle或者沙盒中的文件
>UIWebView可以直接加载一个文件，也可以将二进制数据解析成一个文件并加载出来

	#pragma mark - 加载文件
	- (void)loadFile
	{
	    NSURL *fileURL = [[NSBundle mainBundle] URLForResource:@"关于.txt" withExtension:nil];
	    
	    NSURLRequest *request = [NSURLRequest requestWithURL:fileURL];
	    
	    [self.webView loadRequest:request];
	}

	#pragma 以二进制数据的形式加载文件
	- (void)loadDataFile
	    
	    // 应用场景:加载从服务器上下载的文件,例如pdf,或者word,图片等等文件
	    NSURL *fileURL = [[NSBundle mainBundle] URLForResource:@"iOS 7 Programming Cookbook.pdf" withExtension:nil];
	    
	    NSURLRequest *request = [NSURLRequest requestWithURL:fileURL];
	    // 服务器的响应对象,服务器接收到请求返回给客户端的
	    NSURLResponse *respnose = nil;
	    
	    NSData *data = [NSURLConnection sendSynchronousRequest:request returningResponse:&respnose error:NULL];  //获得二进制数据
	    
	    NSLog(@"%@", respnose.MIMEType);
	    
	    // 先用UTF8解释接收到的二进制数据流
	    [self.webView loadData:data MIMEType:respnose.MIMEType textEncodingName:@"UTF8" baseURL:nil];
	}


>对于apache服务器，和mySQL的搭建，看资料ppt

## NSURLConnection ##
>- 类似与Android中的HttpURLConnection，即在IOS开发中使用这个对象进行网络请求
>- 在网络请求过程中，接收数据的过程实际上是通过NSURLConnectionDataDelegate来实现的；

NSURLConnectionDataDelegate的代理方法：

	服务器开始返回数据,准备工作
	(void)connection:didReceiveResponse:
	收到服务器返回的数据，本方法会被调用多次
	- (void)connection:didReceiveData:
	数据接收完毕，做数据的最后处理
	(void)connectionDidFinishLoading:
	网络连接错误
	- (void)connection:didFailWithError:

>- 使用代理来接收响应结果是有缺陷的
>    - 代理方法较多，比较分散
>    - 要处理一个请求，需要在很多地方编写代码
>    - 不利于逻辑实现、代码编写、调试、维护以及扩展
>    - 尤其当存在多个请求时会变得非常麻烦


使用范例：


	@interface ViewController () <NSURLConnectionDataDelegate>  //遵守协议
	@property (nonatomic, strong) NSMutableData *data;
	@end

	@implementation ViewController
	- (void)viewDidLoad
	{
	    [super viewDidLoad];
		[self logon]
	}
	
	- (void)logon
	{
	    NSURL *url = [NSURL URLWithString:@"http://localhost/login.php"];
	    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url cachePolicy:NSURLRequestUseProtocolCachePolicy timeoutInterval:10.0f];
	    
	    request.HTTPMethod = @"POST";// 请求方式
	    NSString *body = @"username=suisin&password=123";
	    request.HTTPBody = [body dataUsingEncoding:NSUTF8StringEncoding];  //对请求体进行编码
	    
	    NSURLConnection *connection = [[NSURLConnection alloc] initWithRequest:request delegate:self];  
	    [connection start];  //建立连接， 发送请求
	}
	
	#pragma mark - 代理方法
	#pragma mark 接收到服务器的响应
	- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response
	{
	    // 准备工作
	    if (!self.data) {
	        self.data = [NSMutableData data];
	    } else {
	        self.data = nil;
	    }
	}
	
	#pragma mark 收到数据（有可能是一部分）
	//这个方法会被持续调用
	- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
	{
	    [self.data appendData:data];  //接收返回的数据
	}
	
	- (void)connectionDidFinishLoading:(NSURLConnection *)connection
	{
	    // 数据后续处理
	    NSString *result = [[NSString alloc] initWithData:self.data encoding:NSUTF8StringEncoding];  
	    NSLog(@"%@", result);
	}
	
	- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error
	{
	    NSLog(@"%@", error.localizedDescription);
	}
	
	@end



