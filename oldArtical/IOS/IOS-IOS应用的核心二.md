title: IOS-IOS应用的核心二
date: 2016/3/9 8:47:25     
categories: IOS
---

# IOS-IOS应用的核心二 #

接着上一节说。

## IOS应用项目 ##

- Xcode中创建项目（基于Xcode6.4）
> - 一般创建项目的步骤
>     - Create a new project -> ios application -> single view application
>     - 创建好的项目中，已有
>         - AppDelegate, ViewController, Main.storyboard,等文件
>     - Xocde会帮助我们，自动将 AppDelegate设置为应用程序的代理
>     - ViewController为UIWindow的rootViewController
>     - Main.storyboard为程序的Main Interface，并且ViewController为初始化ViewController(箭头指向他)

- BundleIdentifier 与 CompanyIdentifer
>- BundleIdentifier: app的唯一表示， 类似Android中的唯一包名
>- CompanyIdentifer： 公司的唯一标识
>- BundleIdentifier = CompanyIdentifer + ProjectName


- IOS中常见项目文件
> - Info.plist
>     - 本质就是xml文件， 类似android中的AndroidManifest.xml
>     - "工程名-Info.plist"是工程的全局配置文件
>     - 这个文件在Xcode中有对应的图形化界面，我们可以直接通过图形化界面修改应用的配置
>     - 在旧版本Xcode创建的工程中，这个配置文件的名字就叫“Info.plist”
>     - 项目中其他Plist文件不能带有“Info”这个字眼，不然会被错认为是传说中非常重要的“Info.plist”
>     - 项目中还有一个InfoPlist.strings的文件，跟Info.plist文件的本地化相关
>     - 其常见属性如下

	常见属性
	Localiztion native development region(CFBundleDevelopmentRegion)-本地化相关
	
	Bundle display name(CFBundleDisplayName)-程序安装后显示的名称,限制在10－12个字符，如果超出，将被显示缩写名称
	
	Icon file(CFBundleIconFile)-app图标名称,一般为Icon.png
	
	Bundle version(CFBundleVersion)-应用程序的版本号，每次往App Store上发布一个新版本时，需要增加这个版本号
	
	Main storyboard file base name(NSMainStoryboardFile)-主storyboard文件的名称
	
	Bundle identifier(CFBundleIdentifier)-项目的唯一标识，部署到真机时用到


> - .pch文件
>     - 项目的Supporting files文件夹下面有个“工程名-Prefix.pch”文件，也是一个头文件
>     - pch头文件的内容能被项目中的其他所有源文件共享和访问
>     - 一般在pch文件中定义一些全局的宏
>     - 在pch文件中添加下列预处理指令，然后在项目中使用Log(…)来输出日志信息，就可以在发布应用的时候，一次性将NSLog语句移除（在调试模式下，才有定义DEBUG）
>     - 这个文件常见的内容如下

	/**
	 pch文件的作用:
	 1.存放一些全局的宏(整个项目中都用得上的宏)
	 2.用来包含一些全部的头文件(整个项目中都用得上的头文件)
	 3.能自动打开或者关闭日志输出功能
	 */
	#import <Availability.h>
	#ifndef __IPHONE_5_0
	#warning "This project uses features only available in iOS SDK 5.0 and later."
	#endif
	
	/************__OBJC__BEGIN************/
	// 里面的所有内容只能用到.m文件中或者.mm  .m文件中隐藏了__OBJC__这个宏
	#ifdef __OBJC__
	
	#import <UIKit/UIKit.h>
	#import <Foundation/Foundation.h>
	#import "MJPerson.h"
	
	//自定义LOG函数
	#ifdef DEBUG  // 调试阶段
	#define MJLog(...) NSLog(__VA_ARGS__)
	#else // 发布阶段
	#define MJLog(...)
	#endif
	
	#define ABC 10
	
	#endif
	/************__OBJC__END************/
	
	
	/**
	 *  外面的所有东西,整个项目共享
	 */
	#define Name 10

>- storyboard文件
>    - 类似与android中的Activity设置的ContentView
>    - 用于描述一个整的软件界面
>    - 可以使用Interface Builder来编辑它
>    - 在Interface Builder中通过UIViewController的Class属性来绑定界面对应的UIViewController对象
>    - 本质是xml文件

>- xib文件
>    - 效果和storyboard文件差不多，用来描述局部软件界面
>    - 主要用来自定义View
>    - 一个Xib文件中可以描述多个局部界面
>    - xib与nib的关系
>        - xib是面向开发人员的文件格式，当应用运行在手机上时，xib就会变成nib
>    - xib文件本质也是xml文件
>    - 读取xib文件


	//方式1
	//装载一个cell， "MJTGCell"为xib文件的名字， owner：  options：表示返回这个xib文件中的所有view（返回一个数组）
	cell = [[NSBundle mainBundle] loadNibNamed:@"MJTGCell" owner:nil options:nil][0];

	//方式2， 等同于方式1
	UINib* nib = [UINIb nibWithNibName:@"MJTGCell" bundle:[NSBundle mainBundle]];
	cell = [nib instantiateWithOwner:nil options:nil];


>- instancetype返回类型
>    - 许多方法的会徽类型都是他，而不是id
>    - 它在类型表示上跟id一样，可以表示任何类型对象
>    - 它只能用在返回值类型上，不能像id一样用在参数类型上
>    - 编译器会检测instancetype的真实类型
>    
>- Bundle
>    - 可以说一个NSBundle代表一个文件夹
>    - ios下最后所有的资源文件基本都会放到一个文件夹中(mainBundle可以来访问这个文件夹)
>    - 利用mainBundle就可以访问手机中的任何资源([NSBunble mainBundl])
	