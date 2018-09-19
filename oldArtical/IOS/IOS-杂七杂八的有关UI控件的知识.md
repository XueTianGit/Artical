title: IOS-杂七杂八的有关UI控件的知识
date: 2016/3/9 8:51:22               
categories: IOS
---

## IOS-杂七杂八的有关UI控件的知识 ##

>- 本文所记，没有什么组合性，学到一点就记一点 

## 琐碎 ##
>- init方法内部会调用 initWithFrame
>    - 因此，UI控件的初始化应在initWithFrame方法中进行
>- 一般自定义View时，初识化方法只负责添加子控件， layoutSubViews则负责给子控件设置frame
>- initialize这个方法只会调用一次，是类初始化方法，
>- 去除图标的玻璃质感效果
>    -  xcode5： 在Interface Builder中选中 IOS icon is  pre-redered
>- UIBarButtonItem下可以包装许多View做为其子控件， 例如一个Button

- initwithCoder: 与 awakeFromNib：
>- initwithCode:是只要从文件中创建一个对象，就调用这个方法
>- awakeFromNib： 很明显，仅限于Xib文件
>- 调用顺序是 initwithCode -> awakeFromNib

- 自定义控件应重写的方法
>- initwithCoder: 别人重文件中创建
>- initWithFrame: 直接alloc和init

- 启动图片的宽高决定了你的应用的宽高  



## UIButton ##
>- 四种状态
>    - noraml, highlighted : 这两种状态是由用户触发
>    - selected, disable: 一般由代码触发
>- 当按下按钮时 [button setHighlighted:(BOOL)highlighted] 这个方法会做很多事情
>    - 因此按钮会出现被高亮的状态，并且如果一直按着按钮就会一直保持这个状态
>    - 有时我们可能希望当按下按钮时就直接快速过度到下一个状态，因此我们可以重写这个方法，并什么都不做
>    - 也不会使按钮变灰
>- 按钮的样式在system下，按下时会变灰，并不会显示我们设置的高亮下的图片

- 按钮内部的UIImageView
>我们知道按钮内部是有一个imageView的，有时候我们可能需要把这个UIImageView的Frame给调整一下来满足我们的需求
方法是：
1）自定义按钮
2）重写 -(CGRect)imageRectForContentRect:(CGRect)contentRect
3) 在这个方法中返回我们想要的Rect

## UIIamgeView ##

>- stroyboard中的stretching属性
>    - 这个属性，可以拉伸图片的某些像素来使图片美观的显示在imageview中
>    - 其下的4个变量的取值一般为： x:0.5, y:0.5, width:0, height:0, 即拉伸一个像素实现整个图片的拉伸
>- 设置UIIamgeView的圆角
>    - 同时设置这两个属性： imageView.layer.conerRadius 和 imageView.layer.clipsToBounds

如何从大图片中裁剪出小图片

	//计算图片对应的像素大小
    CGFloat smallW = bigImage.size.width / 12  * [UIScreen mainScreen].scale;
    CGFloat smallH = bigImage.size.height * [UIScreen mainScreen].scale;
	CGRect smallRect = CGRectMake(x?, h?, smallW, smallH);
	
	// CGImageCreateWithImageInRect只认像素
	//我们在UIKit框架中使用的单位都是点
	CGImageRef smallImage = CGImageCreateWithImageInRect(bigImage.CGImage, smallRect);
