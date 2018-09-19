title: IOS-动画(UIView)
date: 2016/3/9 8:49:11           
categories: IOS
---

# IOS-动画(UIView)#

>- 对于CALayer的动画，它有一个致命缺陷：并不能真正改变控件的位置(属性值)，只是假象
>- UIKit直接将动画集成到UIView类中，当内部的一些属性发生改变时，UIView将为这些改变提供动画支持
>- UIView动画是真正改变了控件的属性(.....)
>- 执行动画所需要的工作由UIView类自动完成，但仍要在希望执行动画时通知视图。
>- 即需要将改变属性的代码放在[UIView beginAnimations:nil context:nil]和[UIView commitAnimations]之间
>- IOS也为我们提供了简便的使用动画的方式:block




## UIView封装的动画 ##

> 常见方法(UIView)解析

	+ (void)setAnimationDelegate:(id)delegate
	设置动画代理对象，当动画开始或者结束时会发消息给代理对象

	+ (void)setAnimationWillStartSelector:(SEL)selector
	当动画即将开始时，执行delegate对象的selector，并且把beginAnimations:context:中传入的参数传进selector

	+ (void)setAnimationDidStopSelector:(SEL)selector
	当动画结束时，执行delegate对象的selector，并且把beginAnimations:context:中传入的参数传进selector

	+ (void)setAnimationDuration:(NSTimeInterval)duration
	动画的持续时间，秒为单位

	+ (void)setAnimationDelay:(NSTimeInterval)delay
	动画延迟delay秒后再开始

	+ (void)setAnimationStartDate:(NSDate *)startDate
	动画的开始时间，默认为now

	+ (void)setAnimationCurve:(UIViewAnimationCurve)curve
	动画的节奏控制,动画的节奏控制，跟CAAnimation的timingFunction属性类似:
	typedef NS_ENUM(NSInteger, UIViewAnimationCurve) {
	    UIViewAnimationCurveEaseInOut,         // slow at beginning and end
	    UIViewAnimationCurveEaseIn,            // slow at beginning
	    UIViewAnimationCurveEaseOut,           // slow at end
	    UIViewAnimationCurveLinear
	};


	+ (void)setAnimationRepeatCount:(float)repeatCount
	动画的重复次数

	+ (void)setAnimationRepeatAutoreverses:(BOOL)repeatAutoreverses
	如果设置为YES,代表动画每次重复执行的效果会跟上一次相反


	- (void)testViewSimpleAnim
	{
	    [UIView beginAnimations:nil context:nil];
		[UIView setAnimationDuration:1];

	    // 动画执行完毕后, 会自动调用self的animateStop方法
	    //    [UIView setAnimationDelegate:self];
	    //    [UIView setAnimationDidStopSelector:@selector(animateStop)];
	    self.myview.center = CGPointMake(200, 300);
	    [UIView commitAnimations];
	}


- UIView封装的专场动画
>方法：
	
	+ (void)setAnimationTransition:(UIViewAnimationTransition)transition forView:(UIView *)view cache:(BOOL)cache
	设置视图view的过渡效果, transition指定过渡类型, cache设置YES代表使用视图缓存，性能较好

	typedef NS_ENUM(NSInteger, UIViewAnimationTransition) {  //效果相对于CALayer来说比较少
	    UIViewAnimationTransitionNone, 
	    UIViewAnimationTransitionFlipFromLeft,
	    UIViewAnimationTransitionFlipFromRight,
	    UIViewAnimationTransitionCurlUp,
	    UIViewAnimationTransitionCurlDown,
	};


## 使用block来更快的完成动画 ##
>API如下：

	+ (void)animateWithDuration:(NSTimeInterval)duration delay:(NSTimeInterval)delay options:(UIViewAnimationOptions)options animations:(void (^)(void))animations completion:(void (^)(BOOL finished))completion
	参数解析:
	duration：动画的持续时间
	delay：动画延迟delay秒后开始
	options：动画的节奏控制
	animations：将改变视图属性的代码放在这个block中
	completion：动画结束后，会自动调用这个block

	//转场动画

	+ (void)transitionWithView:(UIView *)view duration:(NSTimeInterval)duration options:(UIViewAnimationOptions)options animations:(void (^)(void))animations completion:(void (^)(BOOL finished))completion
	参数解析:
	duration：动画的持续时间
	view：需要进行转场动画的视图
	options：转场动画的类型
	animations：将改变视图属性的代码放在这个block中
	completion：动画结束后，会自动调用这个block

	//转场动画的类型
	UIViewAnimationOptionTransitionNone
	UIViewAnimationOptionTransitionFlipFromLeft 
	UIViewAnimationOptionTransitionFlipFromRight   
	UIViewAnimationOptionTransitionCurlUp 
	UIViewAnimationOptionTransitionCurlDown 
	UIViewAnimationOptionTransitionCrossDissolve 
	UIViewAnimationOptionTransitionFlipFromTop 
	UIViewAnimationOptionTransitionFlipFromBottom



## UIImageView的帧动画 ##

>- UIImageView可以让一系列的图片在特定的时间内按顺序显示
>- 相关属性解析:
>    - animationImages：要显示的图片(一个装着UIImage的NSArray)
>    - animationDuration：完整地显示一次animationImages中的所有图片所需的时间
>    - animationRepeatCount：动画的执行次数(默认为0，代表无限循环)
>- 相关方法解析:
>    - -(void)startAnimating; 开始动画
>    - -(void)stopAnimating;  停止动画
>    - -(BOOL)isAnimating;  是否正在运行动画
