title: IOS-Quartz2D API简单入门
date: 2016/3/9 8:47:59       
categories: IOS
---
# IOS-Quartz2D API简单入门#

## 简介 ##
>- Quartz 2D是一个二维绘图引擎，同时支持iOS和Mac系统
>- Quartz 2D能完成的工作
>    - 绘制图形 : 线条\三角形\矩形\圆\弧等
>    - 绘制文字
>    - 绘制\生成图片(图像)
>    - 读取\生成PDF
>    - 截图\裁剪图片
>    - 自定义UI控件
>    - ...
>- 在IOS开发中，Quartz2D的API是纯C语言的
>- Quartz2D的API来自于Core Graphics框架
>    - 数据类型和函数基本都以CG作为前缀


- Quartz2D在iOS开发中的价值
>- iOS中大部分控件的内容都是通过Quartz2D画出来的
>- 因此，我们可以使用Quartz2D来自定义UI控件

- Quartz2D中的一些核心概念
>- 图形上下文(CGContextRef)
>- 图形上下文的作用
>     - 保存绘图信息、绘图状态
>     - 决定绘制的输出目标（绘制到什么地方去？）
>          - 输出目标可以是PDF文件、Bitmap或者显示器的窗口上
>     - 可以说类似android中的Canvas
>- 相同的一套绘图序列，指定不同的Graphics Context，就可将相同的图像绘制到不同的目标上
>- quartz2d中的图形上下文如下图所示：

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOSquartz2d%E4%B8%AD%E7%9A%84%E5%9B%BE%E5%BD%A2%E4%B8%8A%E4%B8%8B%E6%96%87.png)

>- 图形上下文栈
>    - 顾名思义这个栈是用来保存图形上下文的
>    - 当我们在代码中使用 xxxxGetCurrentContext，基本上得到的都是栈顶的图形上下文
>    - 这个栈对于我们的价值就是，我们可以使用它来保存我们某一时刻的图形上下文状态(各种属性)， 即直接将上下文压入栈中
>    - 这样到我们再需要特定上下文时，我们就可从栈中获取我们已经保存的图形上下文

## 自定义View与 Quartz2D##

### 原理 ###
>- 对于Quartz2d我们已经知道：要想使用Quartz2D来绘制东西必须要有图形上下文。
>- 图形上下文决定着绘制的内容输出到什么地方
>- 当我们自定义View时使用的图形上下文是：Layer Graphics 
>    - UIView中有一个layer属性，这个layer就是用来保存已经使用Quartz2d来绘制东西的上下文
>    - UIView的 -(void)drawRect:(CGRect)rec 方法中含有我们需要的上下文layer
>    - IOS中的UI控件之所以能够显示UI界面，本质是因为有layer这个属性

### 使用Quartz2D自定义View的步骤 ###

1. 新建一个类，继承自UIView
2. 实现-(void)drawRect:(CGRect)rect方法，然后在这个方法中
3. 取得跟当前view相关联的图形上下文
4. 绘制相应的图形内容
5. 利用图形上下文将绘制的所有内容渲染显示到view(layer)上面

- 细节问题
>- -(void)drawRect:(CGRect)rec 方法
>    - 这个方法类似与android中的onDraw方法，即你不可以手动调用这个方法
>    - 方法内含有我们(Quartz2d)需要的上下文layer
>    - 它会在以下时机被调用
>        - 当view第一次显示到屏幕上时（被加到UIWindow上显示出来）
>        - 调用view的setNeedsDisplay或者setNeedsDisplayInRect:时
>        - 注意：setNeedsDisplay和setNeedsDisplayInRect:方法调用后，屏幕并不是立即刷新，
>          而是会在下一次刷新屏幕的时候把绘制的内容显示出来。
>        

- 自定义View的一般性代码
>一般我们会在 -(void)drawRect:(CGRect)rect,做如下事情

	- (void)drawRect:(CGRect)rect
	{
	
	    //1.获得图形上下文
	    CGContextRef ctx = UIGraphicsGetCurrentContext();  //获得 LayerGraphics上下文
	     
		//2.在这个图形上下文中绘制我们所需的图形
		//TODO
	 
	    //3.渲染显示到view上面
	    CGContextStrokePath(ctx);
	}

## Quartz2D API入门 ##
>- 使用Quartz2D主要就是来绘制一些我们想要的图形来满足我们自定义View的需要
>- 使用API时我们还需知道在Quartz2D中的坐标系零点是左下角，但是Core Graphics框架对于大部分API已经帮我们封装成了IOS的环境

- 与图形上下文相关的操作
>- 

	//1.1 在-(void)drawRect:(CGRect)rect方法中，有一个基于layer的上下文，因此我们可以直接这样做
		    CGContextRef ctx = UIGraphicsGetCurrentContext(); //获得LayerGraphics上下文

	//1.2 开启一个基于位图(bitmap)上下文， 即把所有的东西需要绘制到一张新的图片上去
	    // param1: size : 新图片的尺寸
	    // param2：opaque : YES : 不透明,  NO : 透明
	    // 这行代码过后.就相当于常见一张新的bitmap,也就是新的UIImage对象 
	    UIGraphicsBeginImageContextWithOptions(image.size, NO, 0.0);

		//从上下文中取得制作完毕的UIImage对象
   		UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();

		//结束上下文， 自己开启的上下文应记得关闭
   		UIGraphicsEndImageContext();
    //1.3 其他的上下文操作类似

	//1.4 上下文与图形上下文栈
	    // 将ctx拷贝一份放到栈中
   		CGContextSaveGState(ctx); 

	    // 将栈顶的上下文出栈,替换ctx1上下文
    	CGContextRestoreGState(ctx1);



- 与图形上下文相关的操作
>- 设置绘制图形时的环境

	    CGContextSetLineWidth(ctx, 10);  // 设置上下文线段宽度(影响整个上下文的图形的线宽)
		CGContextSetLineCap(ctx, kCGLineCapRound); //设置线段头尾部的样式（直角，还是圆角？）
		CGContextSetLineJoin(ctx, kCGLineJoinRound);  //设置线段转折点的样式（急转弯还是圆弧？）
		
		//设置颜色
		CGContextSetRGBStrokeColor(ctx, 1, 0, 0, 1);  //可以理解为设置画笔的颜色  后面4个参数代表ARGB,取值范围为0-1

	    // set : 同时设置为实心和空心颜色
	    // setStroke : 设置空心颜色
	    // setFill : 设置实心颜色
	    [[UIColor whiteColor] set];
	  

- 绘制图形相关API
>- 绘制图形的思想: 其实我们就是在拼接我们所要绘制图形的路径，最后把他们按照我们的意愿填充我们想要的色彩，(画画不就是这样嘛)
>- 在使用Quartz2d画图时我们可以想象我们手中有一个画笔

	//1. 利用点线来拼接图形
	CGContextMoveToPoint(ctx, 10, 10);  //抬笔， 即画画的第一个点
    CGContextAddLineToPoint(ctx, 100, 100);  //以当前画笔所处的点为起点，画线
    CGContextAddLineToPoint(ctx, 200, 200);
		//1.1 贝塞尔曲线
		CGContextAddQuadCurveToPoint(ctx, controlX, controlY, endX, endY);  //以当前点为起始点，画一条贝塞尔曲线
		
		//1.2 使用CGMutablePathRef 来画线
   		CGMutablePathRef linePath = CGPathCreateMutable();
		CGPathMoveToPoint(linePath, NULL, 0, 0);
 		CGPathAddLineToPoint(linePath, NULL, 100, 100);	

  	    CGMutablePathRef circlePath = CGPathCreateMutable();
   	    CGPathAddArc(circlePath, NULL, 150, 150, 50, 0, M_PI * 2, 0);

 	    CGContextAddPath(ctx, linePath); //添加路径到上下文
		CGContextAddPath(ctx, circlePath);

        CGContextStrokePath(ctx);

	    CGPathRelease(linePath);  //由于非OC对象，因此需要手动释放内存
        CGPathRelease(circlePath);



	//2. 画图形
	    // 2.1 画圆弧
	    // param2,3：x\y : 圆心
	    // param4: radius : 半径
	    // param5:startAngle : 开始角度
	    // param6:endAngle : 结束角度
	    // param7:clockwise : 圆弧的伸展方向(0:顺时针, 1:逆时针)
	    CGContextAddArc(ctx, 100, 100, 50, M_PI_2, M_PI, 0);
		/2.2 画椭圆
		CGContextAddEllipseInRect(ctx, CGRectMake(50, 10, 100, 100));  //可以利用这个API来画圆

		//2.3 画矩形
		CGContextAddRect(ctx, CGRectMake(10, 10, 150, 100));

	//3， 画图片
		[image drawAtPoint:CGPointMake(100, 100)]; //使用OC的API

	//3. 结束图形的绘制
    CGContextClosePath(ctx); //合并路径，即以当前所画的线，按照最简单的方式把线给闭合
    CGContextStrokePath(ctx); //空心填充绘制的图形，即只描绘图形中的线
    CGContextFillPath(ctx);   //实心填充绘制的图形
 
- 矩阵操作与图片裁剪
>- 矩阵操作是对整个上下文整体进行操作
>    - 相关代码应放在画图前执行
>- 我们可以把一张图片裁剪为我们想要的样子
>    - 如果使用Bitmap Graphics Context， 我们还可以获得我们裁剪后的图片
>    - 原理
>        - 先画出一个我们目标图片的形状
>        - 把上下文裁剪成这个形状
>        - 把源图片放到上下文上
>            - 可以利用Bitmap Graphics Context把图片给导出来（生成UIImageView对象）

	//矩阵操作
	CGContextRotateCTM(ctx, M_PI_4 * 0.3);  //旋转
    CGContextScaleCTM(ctx, 0.5, 0.5);      //缩放
    CGContextTranslateCTM(ctx, 0, 150);    //位移
	
	//图片裁剪	
		//TODO1： 画出目标图片的形状
	
		//第二步： 裁剪上下文
		CGContextClip(ctx); 
	    CGContextFillPath(ctx);
	
		//第三步：把源图片放到上下文上
		[image drawAtPoint:CGPointMake(100, 100)];
		
		//在Bitmap Graphics Context中取图
		 UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
		//写出文件
	    NSData *data = UIImagePNGRepresentation(newImage);
	    NSString *path = [[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject] stringByAppendingPathComponent:@"new.png"];
	    [data writeToFile:path atomically:YES];

	

	

	
	
