title: IOS-调用系统资源
date: 2016/3/9 8:51:22               
categories: IOS
---


# IOS-调用系统资源 #

>- iOS中的很多小功能都是非常简单的，几行代码就搞定了，比如打电话、打开网址、发邮件、发短信等
>- 由于大部分些功能都是应用间的操作，因此依赖与UIApplication对象

## 打电话 ##
>打电话共有3种方式，主要区别就是在电话结束后是否可以返回跳转前的应用

方式一：

	//直接跳到拨号界面
	NSURL *url = [NSURL URLWithString:@"tel://10010"];
	[[UIApplication sharedApplication] openURL:url];

但是这种方式有一个缺点是：不会自动回到原应用，直接停留在通话记录界面

方式二：

	NSURL *url = [NSURL URLWithString:@"telprompt://10010"];
	[[UIApplication sharedApplication] openURL:url];

缺点：因为是私有API，所以可能不会被审核通过

方式三：创建一个UIWebView来加载URL，拨完后能自动回到原应用

	if (_webView == nil) {
	    _webView = [[UIWebView alloc] initWithFrame:CGRectZero];  //这个webView并不需要用来显示
	}
	[_webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"tel://10010"]]];

使用这种方式拨完后能自动回到原应用


## 发短信 ##

方式一：

	//直接跳到发短信界面，但是不能指定短信内容，而且不能自动回到原应用
	NSURL *url = [NSURL URLWithString:@"sms://10010"];
	[[UIApplication sharedApplication] openURL:url];

方式二：
>- 使用MessageUI框架，这个框架可以使我们对短信进行详细编辑

	#import <MessageUI/MessageUI.h>

	//显示发短信的控制器
	MFMessageComposeViewController *vc = [[MFMessageComposeViewController alloc] init];
	// 设置短信内容
	vc.body = @"吃饭了没？";
	// 设置收件人列表
	vc.recipients = @[@"10010", @"02010010"];  //可以设置多个收件人
	// 设置代理
	vc.messageComposeDelegate = self;  //准备回调
	 
	// 显示控制器
	[self presentViewController:vc animated:YES completion:nil];  //modal出来

	//代理方法，当短信界面关闭的时候调用，发完后会自动回到原应用
	//这个函数的名称是Apple推荐的，以便我们接收回调参数
	- (void)messageComposeViewController:(MFMessageComposeViewController *)controller didFinishWithResult:(MessageComposeResult)result
	{
	    // 关闭短信界面
	    [controller dismissViewControllerAnimated:YES completion:nil];
	    
	    if (result == MessageComposeResultCancelled) {
	        NSLog(@"取消发送");
	    } else if (result == MessageComposeResultSent) {
	        NSLog(@"已经发出");
	    } else {
	        NSLog(@"发送失败");
	    }
	}

## 发邮件 ##

方式一：

	用自带的邮件客户端，发完邮件后不会自动回到原应用
	NSURL *url = [NSURL URLWithString:@"mailto://10010@qq.com"];
	[[UIApplication sharedApplication] openURL:url];


方式二：跟发短信的第2种方法差不多，只不过控制器类名叫做：MFMailComposeViewController

	//邮件发送后的代理方法回调，发完后会自动回到原应用
	- (void)mailComposeController:(MFMailComposeViewController *)controller didFinishWithResult:(MFMailComposeResult)result error:(NSError *)error
	{
	    // 关闭邮件界面
	    [controller dismissViewControllerAnimated:YES completion:nil];
	    
	    if (result == MFMailComposeResultCancelled) {
	        NSLog(@"取消发送");
	    } else if (result == MFMailComposeResultSent) {
	        NSLog(@"已经发出");
	    } else {
	        NSLog(@"发送失败");
	    }
	}

## 打开其他常见文件 ##

> 如果想打开一些常见文件，比如html、txt、PDF、PPT等，都可以使用UIWebView打开
> 只需要告诉UIWebView文件的URL即可
> 至于打开一个远程的共享资源，比如http协议的，也可以调用系统自带的Safari浏览器：


	NSURL *url = [NSURL URLWithString:@”http://www.baidu.com"];
	[[UIApplication sharedApplication] openURL:url];

## 应用间跳转 ##

> 有时候，需要在本应用中打开其他应用，比如从A应用中跳转到B应用

首先你要知道其他应用的URL地址

>- 一个应用的URL地址可以在Info.plist中配置， key为：URL types
>- 具体格式就是 scheme://identifier

其他应用如果想要跳转到这个应用可以这么做

	//使用UIApplication完成跳转
	NSURL *url = [NSURL URLWithString:@"sx://ios.suixin.cn"];
	[[UIApplication sharedApplication] openURL:url];
 
## 应用评分 ##

> 为了提高应用的用户体验，经常需要邀请用户对应用进行评分
> 应用评分无非就是跳转到AppStore展示自己的应用，然后由用户自己撰写评论

那么如何跳转到AppStore呢？

方法一：
	
	NSString *appid = @"444934666";  //被评分应用的appid
	NSString *str = [NSString stringWithFormat:
	                 @"itms-apps://ax.itunes.apple.com/WebObjects/MZStore.woa/wa/viewContentsUserReviews?type=Purple+Software&id=%@", appid];
	[[UIApplication sharedApplication] openURL:[NSURL URLWithString:str]];

方法二：

	NSString *str = [NSString stringWithFormat:
	                 @"itms-apps://itunes.apple.com/cn/app/id%@?mt=8", appid];
	[[UIApplication sharedApplication] openURL:[NSURL URLWithString:str]];















