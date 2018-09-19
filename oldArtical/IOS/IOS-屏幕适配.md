title: IOS-屏幕适配
date: 2016/3/9 8:51:08              
categories: IOS
---

# IOS-屏幕适配 #

>虽然苹果的机型并没有安卓那么多，但是也是存在屏幕适配问题的。

关于IOS的屏幕：
> - iphone5以下是3.5inch
> - iphone5和iphone5s是4inch
> - iphone6是4.7inch
> - iphone6Plus是5.5inch


## 判断Retina与非Retina ##
> Retina   像素 = 点 * 2

	[UIScreen mainScreen].scale

## autoresizingMask属性 ##
>- 这个属性挺牛的，它主要用于调整子控件相对于父控件的位置(即父控件宽高变化时子控件如何变化)
>- 这个属性可取的枚举值有
>    - UIViewAutoresizingNone                   //
>    - UIViewAutoresizingFlexibleLeftMargin     //代表leftMarging跟随父控件变化而自动调整，取0代表leftMargin是固定距离
>    - UIViewAutoresizingFlexibleWidth          //代表宽度，跟随父控件按相应比例变化
>    - UIViewAutoresizingFlexibleRightMargin    //同leftMargin
>    - UIViewAutoresizingFlexibleTopMargin      //同leftMargin
>    - UIViewAutoresizingFlexibleHeight         //通Width
>    - UIViewAutoresizingFlexibleBottomMargin   //同leftMargin
>- 上面这些属性取1即代表某一方面会自动跟随父控件尺寸变化而变化

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOSautoresizingMask1.png)

>- 如图所示：autoresizingMask属性的取值为： LeftMargin|RightMargin|TopMargin|BottomMargin
>    - 则，上下左右都跟随父控件变化而变化，但是宽高不变

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOSautoresizingMask2.png)

>- 如图所示：autoresizingMask属性的取值为： LeftMargin|RightMargin|TopMargin|BottomMargin |Width|Height
>    - 则，上下左右都跟随父控件变化而变化，并且宽高也跟随父控件变化而变化

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOSautoresizingMask3.png)

>- 如图所示：autoresizingMask属性的取值为： UIViewAutoresizingNone
>    - 则，上下左右都固定，并且宽高固定



