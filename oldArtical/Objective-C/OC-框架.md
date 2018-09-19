title: OC-框架
date: 2016/2/29 15:47:40        
categories: Objective-C
---

# OC-框架 #

- 框架是什么
>框架是一种包类型, 它是一种具有指定布局的目录层次结构，用于把共享的动态库、头文件和资源（图像、声音、nib文件）
>组织进某个单位。从事IOS和MAC OSX开发所需的共享式动态库被包装为框架
>可以看出： OC中的框架类似java中的jar包， 只不过形式是共享式动态库

- 框架
>- 包罗框架：他们是指包含两个或更多个其他框架的框架
>- 使用框架前需要导入： #import<Foundation/Foundation.h>
>- 你直接导入一个框架也没有关系，在编译时Xcode会自动把项目需要的框架添加到项目中
>- Cocoa 和 Cocoa Touch
>    - OS X 程序可以称为Cocoa程序
>    - IOS程序可以称为Cocoa Touch程序
>    - OS X 与ISO具有基本相同的Foundation框架，但是具有不同的UI框架
>        - OS X的UI框架被称为：AppKit
>        - IOS的UI框架被称为：UIKit

- OSX 与 IOS的框架划分
>- OSX（Cocoa框架）
>   - Foundation框架：这个框架包含几乎所有的应用程序都需要的基本独享
>   - AppKit框架：它包含在OS X上构建GUI程序所需的类
>   - Core Data：它与Mac OS X10.4一起推出，它是用于管理持久的对象集合的框架
>   - #import<Cocoa/Cocoa.h>  就直接导入了上面三个框架的头文件
>   - 当然，你也可以分别导入

>- IOS
>与OS X类似，IOS提供类Foundation和Core Data框架，不过，他使用UIKit框架取代了AppKit 

- CoreFoundation框架
>- 它是OC的Foundation框架部分中使用C编写的底层框架。
>- 这个框架中有许多使用C编写的对象类型： CFString, CFArray, CFMutableArray....
>- CoreFoundation对象引用是不透明指针
>    - 它是一个指向结构的指针，该结构的声明放在一个没有提供给用户的私有头文件中
>    - 由于没有头文件，你只能利用一组提供的函数访问结构成员么不能， 不能通过解引用来访问
>    - 用于CoreFoundation对象的指针声明将被typedef所隐藏，它使用类型名称，并在末尾添加“Ref”

	CFMutableArrayRef cfMutableArray = CFArrayCreateMutable(
									CFAllocatorRef alloctor,
									CFIndex capacity,
									const CFArrayCallBacks* callBacks
									);
>- CoreFoundation对象的内存管理
>- 它拥有自己的引用计数的内存管理系统， 与OC的引用计数类似
>- 我们拥有使用 "Create" 或 "Copy"的CF函数而创建的任何对象
>- CFRelease函数可以放弃CF对象的所有权
>- OC的ARC系统不会管理CF对象
>- 利用NULL参数调用CFRetain或CFRelease将导致程序崩溃（nil交给retain和release就不会）

- 免费桥接
>- 一些Foundation类所具有的内存悲剧与对应的CoreFoundation类的内存布局几乎完全相同
>- 因此，为了方便使用，我们可以直接把CF对象强转为Foundation对象
>- 例如： NSString 与 CFStringRef 之间就可以直接强转

- Apple提供的其他框架
>- WebKit， ImageIO， CoreImage， CoreAudio
>- OpenGL：用于执行快速的、硬件加速的3D图形处理的C语言框架
>- OpenAL：开源的音频库
>- OpenCL：一个C语言框架，使用GPU和多核CPU的能力执行高性能的计算。

OC中的框架，可以通过查看：/System/Library/Frameworks的内容