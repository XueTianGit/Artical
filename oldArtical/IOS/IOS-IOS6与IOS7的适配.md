title: IOS-IOS6与IOS7的适配
date: 2016/3/9 8:46:43   
categories: IOS
---

# IOS-IOS6与IOS7的适配 #

- 判断SDK的版本
>- <Availiablity.h> 头文件中有许多与SDK版本，硬件环境相关的宏

	#import <Availiablity.h>
	//判断SDK（xcode）版本
	#ifdef _IPHONE_7_0
		//TODO
	#endif

- 判断当前的IOS版本
	[[UIDevice currentDevice].systemVersion doubleValue]  //拿到系统版本号

## IOS6的控制器与IOS7的控制器的问题 ##

- edgesForExtendedLayout属性
>- 我们都知道，IOS6和IOS7的控制器的Frame是不同的
>    -IOS7是占据这个屏幕
>- 那么这个控制器显示(view)大小到底是谁控制的呢，恩，就是edgesForExtendedLayout属性
>- edgesForExtendedLayout属性可选值有
>    - UIRectEdgeBottom， UIRectEdgeNone，UIRectEdgeLeft.....UIRectEdgeAll
>- IOS7默认值是UIRectEdgeAll，所有IOS7的控制器(view)是填充整个屏幕的
>- 那么这个属性有什么用处呢？
>    - 如果控制器的View不是tableView，scrollView类似的View，那完全没有必要设置为UIRectEdgeAll
>- 这个属性可以在Interface Builder中设置


## 去除IOS6的tableView的margin效果 ##

> 我们知道，IOS7的tableView的每个cell的都是宽都是填充屏幕的，而IOS6cell的宽是和屏幕右一定的margin的
> 因此，如果这里不进行适配的话，那么我们的app在IOS6和IOS7上的效果会有很大的不同。

那么如何适配呢？
>- 首先要知道的是
>    - cell的frame由tableView决定，即tableView获得cell的frame
>    - 我们看到的cell的宽度实际上是其内部contentView的宽度
>    - 重写cell的setFrame可以从根本上改变cell的frame (父控件是依赖于这个方法来获得cell的frame的)
>    - cell中的accessotyView并不属于contentView

综上，我们只需重写cell的setFrame:即可：

	/**
	 *  拦截frame的设置
	 */
	- (void)setFrame:(CGRect)frame
	{
	    if (!iOS7) {   //如果不是IOS7， 即IOS6
	        CGFloat padding = 10;
	        frame.size.width += padding * 2;  //给cell的frame的宽度增加2个padding， 
	        frame.origin.x = -padding;       // 将x坐标向左移动，以调整contentView的位置，获得我们想要的效果
	    }
	    [super setFrame:frame];
	}






