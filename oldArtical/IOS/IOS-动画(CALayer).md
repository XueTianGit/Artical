title: IOS-动画(CALayer)
date: 2016/3/9 8:48:56          
categories: IOS
---

# IOS-动画(CALayer)#

- 引言
>- 就像Android中的动画一样，IOS中的动画类似。
>- 首先动画要有一个作用对象吧，在IOS中这个对象就是UIView
>- 在IOSUIView有两个功能
>    - 显示交互界面： 依赖于其内部的layer
>    - 接收和处理事件： 继承自UIResponder
>- 很明显，我们看到的动画最终UIView是依赖于layer完成的


## Core Animation 框架##

### CALayer ###

#### 与UIView的关系 ####

>- 在iOS中，你能看得见摸得着的东西基本上都是UIView，比如一个按钮、一个文本标签、一个文本输入框、一个图标等等，这些都是UIView
>- 其实UIView之所以能显示在屏幕上，完全是因为它内部的一个图层(layer)
>- 在创建UIView对象时，UIView内部会自动创建一个图层(即CALayer对象)，通过UIView的layer属性可以访问这个层
>     - @property(nonatomic,readonly,retain) CALayer *layer; 
>- 当UIView需要显示到屏幕上时，会调用drawRect:方法进行绘图，并且会将所有内容绘制在自己的图层上，绘图完毕后，系统会将图层拷贝到屏幕上，于是就完成了UIView的显示
>- 换句话说，UIView本身不具备显示的功能，是它内部的层才有显示功能

即在显示界面这件事上，如果我们的UI并不与用户交互，完全可以使用CALayer，使用API类似与UIView

#### CALayer的基本使用 ####

> - 通过操作CALayer对象，可以很方便地调整UIView的一些外观属性，比如：阴影、圆角大小、边框宽度和颜色… …
> - 还可以给图层添加动画，来实现一些比较炫酷的效果


CALayer的属性及基本使用

	@property CGRect bounds; 	//宽度和高度
	@property CGPoint position;  //	位置(默认指中点，具体由anchorPoint决定)
	@property CGPoint anchorPoint; //锚点(x,y的范围都是0-1)，决定了position的含义
	@property CGColorRef backgroundColor;  //	背景颜色(CGColorRef类型)
	@property CATransform3D transform;  //	形变属性
	@property CGColorRef borderColor;  //	边框颜色(CGColorRef类型)
	@property CGFloat borderWidth;   //	边框宽度
	@property CGColorRef borderColor;  //	圆角半径
	@property(retain) id contents;  //	内容(比如设置为图片CGImageRef)  


	//添加一个边框
	iconView.layer.borderWidth = 10;  //边框占的区域是从空间边缘向内部延伸，即添加边框会使空间的可显示内容变小

	//给view添加圆角   

    self.iconView.layer.cornerRadius = 10;
    // 超出主层边框范围的内容都剪掉  (对于imageView来说，剪的就是图片)
    self.iconView.layer.masksToBounds = YES;   //这个是针对imageView等类似控件的， 这是因为imageView显示内容时分为： 主层(layer) + 子层(图片)
	
	//给一个view添加阴影
    // 阴影颜色
    iconView.layer.shadowColor = [UIColor blueColor].CGColor;
    // 阴影偏差
    iconView.layer.shadowOffset = CGSizeMake(20, 20);
    // 阴影不透明度
    iconView.layer.shadowOpacity = 0.5; 


对于CATransform3D transform属性的使用

	//方式1：直接更改  CATransform3D transform; 形变属性
    iconView.layer.transform = CATransform3DMakeScale(1.5, 0.5, 0);  //是view缩放， 3d形式
    iconView.transform = CGAffineTransformMakeRotation(M_PI_4);  //2d
	iconView.layer.transform = CATransform3DMakeRotation(M_PI_4, 0, 0, 1);  //旋转， 

	//方式2：通过keyPath来设置transform

    NSValue *value = [NSValue valueWithCATransform3D:CATransform3DMakeRotation(M_PI_4, 0, 0, 1)];  //沿Z轴旋转
    [self.iconView.layer setValue:value forKeyPath:@"transform"];
    self.iconView.layer.transform = CATransform3DMakeScale(0.5, 2, 0);  //缩放
    [self.iconView.layer setValue:[NSValue valueWithCATransform3D:CATransform3DMakeScale(0.5, 2, 0)] forKeyPath:@"transform"];

	//设置更细致
    [self.iconView.layer setValue:@(M_PI_2) forKeyPath:@"transform.rotation"];    
    [self.iconView.layer setValue:@(-100) forKeyPath:@"transform.translation.x"];      // 可以传递哪些key path, 在官方文档搜索 "CATransform3D key paths"

- position和anchorPoint
>类似与游戏开发中的锚点的概念

	@property CGPoint position;
	用来设置CALayer在父层中的位置
	以父层的左上角为原点(0, 0)

	@property CGPoint anchorPoint;
	称为“定位点”、“锚点”
	决定着CALayer身上的哪个点会在position属性所指的位置       //即锚点要有position重合
	以自己的左上角为原点(0, 0)
	它的x、y取值范围都是0~1，默认值为（0.5, 0.5）



#### 新建图层 ####
>类似与UIView

	- (void)viewDidLoad  
	{
	    [super viewDidLoad];
	
	    // 新建图层
		//CALayer *layer = [[CALayer alloc] init];
	    CALayer *layer = [CALayer layer];  
	    layer.backgroundColor = [UIColor redColor].CGColor;  
	    layer.bounds = CGRectMake(0, 0, 100, 100);
	    layer.position = CGPointMake(200, 100);
	    layer.cornerRadius = 10;
	    layer.masksToBounds = YES;
	    layer.contents = (id)[UIImage imageNamed:@"lufy"].CGImage;
	    [self.view.layer addSublayer:layer];
	}


#### 隐式动画 ####

>- 每一个UIView内部都默认关联着一个CALayer，我们可用称这个Layer为Root Layer（根层）
>- 所有的非Root Layer，也就是手动创建的CALayer对象，都存在着隐式动画
>- 什么是隐式动画？
>    - 当对非Root Layer的部分属性进行修改时，默认会自动产生一些动画效果
>    - 而这些属性称为Animatable Properties(可动画属性)

几个常见的Animatable Properties：
bounds：用于设置CALayer的宽度和高度。修改这个属性会产生缩放动画
backgroundColor：用于设置CALayer的背景色。修改这个属性会产生背景色的渐变动画
position：用于设置CALayer的位置。修改这个属性会产生平移动画


- 如何关闭隐士动画呢？
>可以通过动画事务(CATransaction)关闭默认的隐式动画效果

	[CATransaction begin];
	[CATransaction setDisableActions:YES];
	self.myview.layer.position = CGPointMake(10, 10);  //不会产生隐士动画的效果了
	[CATransaction commit];

### UIView和CALayer的选择 ###

从上面的使用可以看出，通过CALayer，就能做出跟UIImageView一样的界面效果
既然CALayer和UIView都能实现相同的显示效果，那究竟该选择谁好呢？

>- 其实，对比CALayer，UIView多了一个事件处理的功能。也就是说，CALayer不能处理用户的触摸事件，而UIView可以
>- 所以，如果显示出来的东西需要跟用户进行交互的话，用UIView；如果不需要跟用户进行交互，用UIView或者CALayer都可以
>- 当然，CALayer的性能会高一些，因为它少了事件处理的功能，更加轻量级


### 核心动画(Core Animation)###

>Core Animation是一组非常强大的动画处理API，使用它能做出非常炫丽的动画效果，而且往往是事半功倍，
>使用它需要先添加QuartzCore.framework和引入对应的框架<QuartzCore/QuartzCore.h> （xcode5之后已经默认引入了这个框架）

>- CALayer中很多属性都可以通过CAAnimation实现动画效果包括：opacity、position、transform、bounds、contents等(可以在API文档中搜索：CALayer Animatable Properties)
>- 通过调用CALayer的addAnimation:forKey增加动画到层(CALayer)中，这样就能触发动画了。通过调用removeAnimationForKey可以停止层中的动画
>- Core Animation的动画执行过程都是在后台操作的，不会阻塞主线程

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOSCAAnimation%E7%BB%A7%E6%89%BF%E7%BB%93%E6%9E%84.png)


- CAAnimation

>- 所有动画对象的父类，负责控制动画的持续时间和速度，是个抽象类，不能直接使用，应该使用它具体的子类
>- 属性解析：(红色代表来自CAMediaTiming协议的属性)
>    - duration：动画的持续时间
>      repeatCount：动画的重复次数
>      repeatDuration：动画的重复时间
>      removedOnCompletion：默认为YES，代表动画执行完毕后就从图层上移除，图形会恢复到动画执行前的状态。
>      如果想让图层保持显示动画执行后的状态，那就设置为NO，不过还要设置fillMode为kCAFillModeForwards
>      fillMode：决定当前对象在非active时间段的行为.比如动画开始之前,动画结束之后
>      beginTime：可以用来设置动画延迟执行时间，若想延迟2s，就设置为CACurrentMediaTime()+2，CACurrentMediaTime()为图层的当前时间
>      timingFunction：速度控制函数，控制动画运行的节奏
>      delegate：动画代理

- CAPropertyAnimation
>- 是CAAnimation的子类，也是个抽象类，要想创建动画对象，应该使用它的两个子类：CABasicAnimation和CAKeyframeAnimation
>- 属性解析：
>     keyPath：通过指定CALayer的一个属性名称为keyPath(NSString类型)，并且对CALayer的这个属性的值进行修改，达到相应的动画效果。
>     比如，指定@”position”为keyPath，就修改CALayer的position属性的值，以达到平移的动画效果

- CABasicAnimation
>- CAPropertyAnimation的子类
>- 属性解析:
>    fromValue：keyPath相应属性的初始值
>    toValue：keyPath相应属性的结束值
>- 随着动画的进行，在长度为duration的持续时间内，keyPath相应属性的值从fromValue渐渐地变为toValue
>- 如果fillMode=kCAFillModeForwards和removedOnComletion=NO，那么在动画执行完毕后，图层会保持显示动画执行后的状态。
>  但在实质上，图层的属性值还是动画执行前的初始值，并没有真正被改变。

	- (void)testTransform
	{
	    // 1.创建动画对象
	    CABasicAnimation *anim = [CABasicAnimation animation];
	    
	    // 2.设置动画对象
	    // keyPath决定了执行怎样的动画, 调整哪个属性来执行动画
	    //    anim.keyPath = @"transform.rotation";
	    //    anim.keyPath = @"transform.scale.x";
		//    anim.keyPath = @"bounds";
   		//    anim.keyPath = @"position";
	    anim.keyPath = @"transform.translation.x";
	    anim.toValue = @(100);  //fromValue 默认为当前值
	    //    anim.toValue = [NSValue valueWithCGPoint:CGPointMake(100, 100)];
	    anim.duration = 2.0;
	    
		//动画结束后保持形态
	    anim.removedOnCompletion = NO;
	    anim.fillMode = kCAFillModeForwards;
	    
	    // 3.添加动画
	    [self.myLayer addAnimation:anim forKey:nil];  
	}

- CAKeyframeAnimation

>- CApropertyAnimation的子类，跟CABasicAnimation的区别是：CABasicAnimation只能从一个数值(fromValue)变到另一个数值(toValue)，
>   而CAKeyframeAnimation会使用一个NSArray保存这些数值
>- 属性解析：
>     - values：就是上述的NSArray对象。里面的元素称为”关键帧”(keyframe)。动画对象会在指定的时间(duration)内，依次显示values数组中的每一个关键帧
>       path：可以设置一个CGPathRef\CGMutablePathRef,让层跟着路径移动。path只对CALayer的anchorPoint和position起作用。如果你设置了path，那么values将被忽略
>       keyTimes：可以为对应的关键帧指定对应的时间点,其取值范围为0到1.0,keyTimes中的每一个时间值都对应values中的每一帧.当keyTimes没有设置的时候,各个关键帧的时间是平分的
>- CABasicAnimation可看做是最多只有2个关键帧的CAKeyframeAnimation

	- (void)testMove
	{
	    CAKeyframeAnimation *anim = [CAKeyframeAnimation animation];
	    
	    anim.keyPath = @"position";
	    anim.removedOnCompletion = NO;
	    anim.fillMode = kCAFillModeForwards;
	    anim.duration = 2.0;
	    
		//可以通过path对象，
	    CGMutablePathRef path = CGPathCreateMutable();
	    CGPathAddEllipseInRect(path, NULL, CGRectMake(100, 100, 200, 200));
	    anim.path = path;
	    CGPathRelease(path);

		//也可以通过values属性
	    //NSValue *v1 = [NSValue valueWithCGPoint:CGPointZero];
	    //NSValue *v2 = [NSValue valueWithCGPoint:CGPointMake(100, 0)];
	    //NSValue *v3 = [NSValue valueWithCGPoint:CGPointMake(100, 200)];
	    //NSValue *v4 = [NSValue valueWithCGPoint:CGPointMake(0, 200)];
	    //anim.values = @[v1, v2, v3, v4];
	    
	    // 设置动画的执行节奏
	    // kCAMediaTimingFunctionEaseInEaseOut : 一开始比较慢, 中间会加速,  临近结束的时候, 会变慢
	    anim.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
	    anim.delegate = self;
	    
	    [self.redView.layer addAnimation:anim forKey:nil];
	}

- CAAnimationGroup

>- CAAnimation的子类，可以保存一组动画对象，将CAAnimationGroup对象加入层后，组中所有动画对象可以同时并发运行
>- 属性解析：
>    - animations：用来保存一组动画对象的NSArray
>- 默认情况下，一组动画对象是同时运行的，也可以通过设置动画对象的beginTime属性来更改动画的开始时间


- CATransition

>- CAAnimation的子类，用于做转场动画，能够为层提供移出屏幕和移入屏幕的动画效果。iOS比Mac OS X的转场动画效果少一点
>- UINavigationController就是通过CATransition实现了将控制器的视图推入屏幕的动画效果
>- 属性解析:
>    - type：动画过渡类型(效果)
>    - subtype：动画过渡方向
>    - startProgress：动画起点(在整体动画的百分比)
>    - endProgress：动画终点(在整体动画的百分比)


	//过渡效果
	//fade     //交叉淡化过渡(不支持过渡方向) kCATransitionFade
	// push     //新视图把旧视图推出去  kCATransitionPush
 	//moveIn   //新视图移到旧视图上面   kCATransitionMoveIn
	//reveal   //将旧视图移开,显示下面的新视图  kCATransitionReveal
	// cube     //立方体翻滚效果
	// oglFlip  //上下左右翻转效果
	//suckEffect   //收缩效果，如一块布被抽走(不支持过渡方向)
	//rippleEffect //滴水效果(不支持过渡方向)
	//pageCurl     //向上翻页效果
	// cameraIrisHollowOpen  //相机镜头打开效果(不支持过渡方向)

	-(void)testTransition
	{
	   //转场动画
	    CATransition *anim = [CATransition animation];
	    anim.type = @"pageCurl";
		//anim.subtype = kCATransitionFromRight;  //翻页方向
	    anim.duration = 0.5;
	    
		//anim.startProgress = 0.0;  
		//anim.endProgress = 0.5;
	    
	    [self.view.layer addAnimation:anim forKey:nil];	
	}




