title: IOS- 多控制器管理二
date: 2016/3/9 8:45:55  
categories: IOS
---

# IOS- 多控制器管理二 #

## 控制器的数据传递 ##
>- 控制器之间的数据传递主要有2种情况:顺传和逆传
>- 假设有两个控制器A和C

- 顺传
> - 控制器的跳转方向: A -> C
> - 数据的传递方向: A -> C
> - 数据的传递方式:  
>     - 在A的prepareForSegue:sender:方法中根据segue参数取得destinationViewController, 也就是控制器C, 直接给控制器C传递数据
>     - (要在C的viewDidLoad方法中取得数据,来赋值给界面上的UI控件)

- 逆传
> - 控制器的跳转方向: A -> C
> - 数据的传递方向: C -> A
> - 数据的传递方式:  
>     - 让A成为C的代理（应在A跳到C之前完成）
>     - 在C中调用代理方法,通过代理方法的参数传递数据给A


## UITabBarController ##
>跟UINavigationController类似，UITabBarController也可以轻松地管理多个控制器，轻松完成控制器之间的切换，典型例子就是QQ、微信等应用

- 一般使用步骤
>- 初始化UITabBarController
>- 设置UIWindow的rootViewController为UITabBarController
>- 根据具体情况，通过addChildViewController方法添加对应个数的子控制器
>- 添加控制器的方法有两种
>    - -(void)addChildViewController:(UIViewController *)childController;
>    - @property(nonatomic,copy) NSArray *viewControllers;  //直接设置这个数组


- UITabBar
>- 每一个UITabBarController内都会有一个UITabBar（高度为49）
>- 如果UITabBarController有N个子控制器，那么UITabBar内部就会有N个UITabBarButton作为子控件

- UITabBarButton
>- UITabBarButton里面显示什么内容，由对应子控制器的tabBarItem属性决定
>- UITabBarItem有以下属性影响着UITabBarButton的内容
>     - 标题文字
>       @property(nonatomic,copy) NSString *title;
>     - 图标
>       @property(nonatomic,retain) UIImage *image;
>     - 选中时的图标
>       @property(nonatomic,retain) UIImage *selectedImage;
>       在默认情况下，如果没有提供这个图片的话，在选中时会把image变蓝
>     - 提醒数字
>       @property(nonatomic,copy) NSString *badgeValue;

>- 下图是APP主流的框架（微信，QQ等）

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOSApp%E4%B8%BB%E6%B5%81%E6%A1%86%E6%9E%B6.png)








