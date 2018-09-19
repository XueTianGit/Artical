title: IOS-事件处理
date: 2016/3/9 8:51:22               
categories: IOS
---

# IOS-事件处理 #

## IOS系统中的事件 ##

- 简单概述
>- IOS目前支持三种类型的事件：触摸事件、运动事件和远程控制事件
>- 这些事件使用UIEvent来表示
>- 每个事件(UIEvent)都有一个与之关联的事件类型和子类型， 可以通过type和subType属性访问
>    - 触摸事件
>        - ios中的触摸事件是基于多点触摸
>        - 不同的UIKit对象，对触摸手势的处理是不一样的
>    - 运动事件
>        - 当以特定方式移动设备(比如摇摆)时，就会产生运动事件
>        - 运动事件源自设备加速器
>        - 运动事件除了事件类型，子类型和时间戳之外，没有其他状态
>        - 处理事件必须实现 motionBegan:withEvent; motionEnabled:withEvent

- 事件与触摸
>- 在ios中触摸动作是指手指碰到屏幕或在屏幕上移动
>- 不同的手势对应不同的触摸动作
>- 事件是当用户手指触击屏幕及在屏幕上移动时，系统不断发送给应用程序的对象
>- 触摸信息有时间和空间两方面，时间方面的信息称为阶段，被封装在UIEvent中，其实就是开始，移动，结束等阶段 


- 如何处理事件
>- 在iOS中不是任何对象都能处理事件，只有继承了UIResponder的对象才能接收并处理事件。我们称之为“响应者对象”
>- UIApplication、UIViewController、UIView都继承自UIResponder，因此它们都是响应者对象，都能够接收并处理事件

- UIEvent
> 每产生一个事件，就会产生一个UIEvent对象，记录事件产生的时刻和类型

	常见属性如下：
	@property(nonatomic,readonly) UIEventType     type;
	@property(nonatomic,readonly) UIEventSubtype  subtype;
	事件产生的时间
	@property(nonatomic,readonly) NSTimeInterval  timestamp;

> UIEvent还提供了相应的方法可以获得在某个view上面的触摸对象（UITouch）

- UITouch
>- 当用户用一根触摸屏幕时，会创建一个与手指相关联的UITouch对象，一根手指对应一个UITouch对象
>- 当手指离开屏幕时，系统会销毁相应的UITouch对象
>- UITouch的作用
>    - 保存着跟手指相关的信息，比如触摸的位置、时间、阶段
>    - 当手指移动时，系统会更新同一个UITouch对象，使之能够一直保存该手指在的触摸位置

	//UITouch的属性
	触摸产生时所处的窗口
	@property(nonatomic,readonly,retain) UIWindow    *window;
	
	触摸产生时所处的视图
	@property(nonatomic,readonly,retain) UIView      *view;
	
	短时间内点按屏幕的次数，可以根据tapCount判断单击、双击或更多的点击
	@property(nonatomic,readonly) NSUInteger          tapCount;
	
	记录了触摸事件产生或变化时的时间，单位是秒
	@property(nonatomic,readonly) NSTimeInterval      timestamp;
	
	当前触摸事件所处的状态
	@property(nonatomic,readonly) UITouchPhase        phase;
	
	//UITouch的方法
	- (CGPoint)locationInView:(UIView *)view;
	返回值表示触摸在view上的位置
	这里返回的位置是针对view的坐标系的（以view的左上角为原点(0, 0)）
	调用时传入的view参数为nil的话，返回的是触摸点在UIWindow的位置
	
	- (CGPoint)previousLocationInView:(UIView *)view;
	该方法记录了前一个触摸点的位置



## 事件传递与处理 ##

### 事件的传递 ###
>- 事件发生后被封装成UIEvent对象放入事件队列
>- UIApplication对象会将事件从队列顶部取出，通过sendEvent:方法进行派发
>- 派发者一般是当前交互窗口，UIWindow会通过sendEvent:继续派发
>- 在派发过程中通过触碰测试(hitTest:withEvent：)方法来寻找事件的第一响应者，返回true则代表响应事件
>- UIWindow对象以消息的形式将事件发送给第一响应者(一般是UIView)

那么自UIWindow开始分发事件时是怎么做的呢？

- 寻找最合适的响应者

>- 首先，最重要的一点是： 如果父控件不能接收触摸事件，那么子控件就不可能接收到触摸事件
>- 如何找到最合适的控件来处理事件？
>    - 当事件来到UIResponder对象时，先判断自己是否能接收触摸事件？
>    - 判断触摸点是否在自己身上？
>    - 从后往前遍历子控件，重复前面的两个步骤
>    - 如果没有符合条件的子控件，那么就自己最适合处理

>可以看出，在寻找最合适响应者的过程中，子控件有优先处理事件的特权

### 处理事件 ###

找到了事件的最佳处理者，那么它怎么处理事件呢？

>- 如果一个对象想要处理事件，那么必须继承自UIResponder
>- 实现UIResponder处理事件的接口方法

	UIResponder内部提供了以下方法来处理事件
	触摸事件
	- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;
	- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;
	- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event;
	- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event;
	
	加速计事件
	- (void)motionBegan:(UIEventSubtype)motion withEvent:(UIEvent *)event;
	- (void)motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event;
	- (void)motionCancelled:(UIEventSubtype)motion withEvent:(UIEvent *)event;
	
	远程控制事件
	- (void)remoteControlReceivedWithEvent:(UIEvent *)event;

>- 对于触摸
>- 4个触摸事件处理方法中，都有NSSet *touches和UIEvent *event两个参数
>     - 一次完整的触摸过程中，只会产生一个事件对象，4个触摸方法都是同一个event参数
>       如果两根手指同时触摸一个view，那么view只会调用一次touchesBegan:withEvent:方法，touches参数中装着2个UITouch对象
>       如果这两根手指一前一后分开触摸同一个view，那么view会分别调用2次touchesBegan:withEvent:方法，并且每次调用时的touches参数中只包含一个UITouch对象
>       根据touches中UITouch的个数可以判断出是单点触摸还是多点触摸


#### 处理事件中的响应者链 ####

>- 在找到事件的最佳处理者后，开始调用UIResponder的touches方法等方法
>- 这些touches方法的默认做法是将事件顺着响应者链条向上传递，将事件交给下一个响应者进行处理
>- 即如果你不覆写touches方法，那么事件会继续沿响应者链进行传递

- 响应者链 
>- UIResponder中有一个这个方法: -(UIResponder*)nextResponder
>- 该方法返回下一个响应者
>- 该方法默认是实现是将view的superview或者控制器作为下一个响应者


![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E5%93%8D%E5%BA%94%E8%80%85%E9%93%BE.png)

根据图可以分析得：
>- 如果view的控制器存在，就传递给控制器；如果控制器不存在，则将其传递给它的父视图
>- 在视图层次结构的最顶级视图，如果也不能处理收到的事件或消息，则其将事件或消息传递给window对象进行处理
>- 如果window对象也不处理，则其将事件或消息传递给UIApplication对象
>- 如果UIApplication也不能处理该事件或消息，则将其丢弃



## 监听触摸事件的其他做法 ##

>- 如果想监听一个view上面的触摸事件，之前的做法是
>    - 自定义一个view
>    - 实现view的touches方法，在方法内部实现具体处理代码
>- 但是通过touches方法监听view触摸事件，有很明显的几个缺点
>    - 必须得自定义view
>    - 由于是在view内部的touches方法中监听触摸事件，因此默认情况下，无法让其他外界对象监听view的触摸事件
>    - 不容易区分用户的具体手势行为

iOS 3.2之后，苹果推出了手势识别功能（Gesture Recognizer），在触摸事件处理方面，大大简化了开发者的开发难度

### UIGestureRecognizer ###

>- 即手势识别器，利用UIGestureRecognizer，能轻松识别用户在某个view上面做的一些常见手势
>- UIGestureRecognizer是一个抽象类，定义了所有手势的基本行为，使用它的子类才能处理具体的手势
>    - UITapGestureRecognizer(敲击)
>    - UIPinchGestureRecognizer(捏合，用于缩放)
>    - UIPanGestureRecognizer(拖拽)
>    - UISwipeGestureRecognizer(轻扫)
>    - UIRotationGestureRecognizer(旋转)
>    - UILongPressGestureRecognizer(长按)


### 手势识别器的使用步骤 ###

> 每一个手势识别器的用法都差不多，比如UITapGestureRecognizer的使用步骤如下

	//1. 创建手势识别器对象
	UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] init];
	
	//2. 设置手势识别器对象的具体属性
	//连续敲击2次
	tap.numberOfTapsRequired = 2;
	//需要2根手指一起敲击
	tap.numberOfTouchesRequired = 2;
	
	//3.添加手势识别器到对应的view上
	[self.iconView addGestureRecognizer:tap];
	
	//4. 监听手势的触发
	[tap addTarget:self action:@selector(tapIconView:)];  //

	//可以使用removeGestureRecognizer: 来移除手势。

>- 下面来看几个比较特别的手势

旋转手势

	//下面这两个方法实现了对view的选择
	- (void)testRotate
	{
	    UIRotationGestureRecognizer *rotate = [[UIRotationGestureRecognizer alloc] initWithTarget:self action:@selector(rotateView:)];
	    [self.iconView addGestureRecognizer:rotate];   //self为控制器
	}

	- (void)rotateView:(UIRotationGestureRecognizer *)recognizer
	{
		//recognizer.view， 即添加手势的那个view
		//recognizer.rotation 即旋转的角度， 注意的是这个属性的值会随旋转角度不断累加的，因此每次旋转view完之后，应立刻清零
	    recognizer.view.transform = CGAffineTransformRotate(recognizer.view.transform, recognizer.rotation);
	    recognizer.rotation = 0; 
	}

缩放手势

	//代码与旋转手势类似
	- (void)testPinch
	{
	    UIPinchGestureRecognizer *pinch = [[UIPinchGestureRecognizer alloc] initWithTarget:self action:@selector(pinchView:)];
	    [self.iconView addGestureRecognizer:pinch];
	}
	
	- (void)pinchView:(UIPinchGestureRecognizer *)pinch
	{
	    pinch.view.transform = CGAffineTransformScale(pinch.view.transform, pinch.scale, pinch.scale);
	    pinch.scale = 1; // 立刻清除
	}


多个手势的共同识别

>- 默认情况下，是不允许多个手势共同识别的
>- 我们可以这样做， 来实现多个手势的共同识别

	//以上面的两个手势为例，我们稍微补全一下代码，来完成多个手势的识别
	//1. 在创建手势时设置代理
    rotate.delegate = self;  //选择手势的代理为控制器
    pinch.delegate = self;
	
	//2. 覆写这个方法，并返回YES
	- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer
	{
	    return YES;
	}
























 


