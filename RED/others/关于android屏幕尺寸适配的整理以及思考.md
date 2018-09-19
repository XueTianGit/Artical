>一直以来android屏幕尺寸相关的东西我都很薄弱，什么dpi, ppi, 英寸我都比较疑惑，本文主要是理清概念，理解头条的屏幕适配原理，以为目前我工作是如何做UI适配的。

## 一些基础概念

### 屏幕尺寸

屏幕尺寸指屏幕的对角线的长度，单位是英寸，1英寸=2.54厘米。这个值是利用手机屏幕的长和宽，然后利用勾股定理，就可以算出斜边的长了。

### 屏幕像素密度

屏幕像素密度，即每英寸屏幕所拥有的像素数，英文简称ppi, 屏幕像素密度与屏幕尺寸和屏幕分辨率有关，在单一变化条件下，屏幕尺寸越小、分辨率越高，像素密度越大，反之越小。

那dpi又是什么呢？ 其实对Android而言，DPI等价于PPI。 具体解释可以参考这篇文章: 
https://blog.csdn.net/u010134087/article/details/54926403

### 屏幕分辨率
屏幕分辨率是指在横纵向上的像素点数，单位是px，1px=1个像素点

我们可以看一下下面这两张图,就可以理清上面三个概念了:

![newppi.png](https://upload-images.jianshu.io/upload_images/2934684-15207db5538374ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![ppi.jpg](https://upload-images.jianshu.io/upload_images/2934684-70503de74d7e77c6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## Android中的屏幕尺寸相关概念

### dp

dp是一种密度无关像素，对应于160dpi下像素的物理尺寸。它与px的关系是 : px = dp * (dpi / 160)。

>sp的概念与dp类似，是用来表示字体大小的一个单位。

即在160dpi下，1dip=1px，如果320dpi，则1dip=2px，以此类推。这里延伸出我们常说的一个密度,其实这里就可以理解为:  (dpi / 160)。
常见的密度：

xxhdpi:   3.0
xhdpi：  2.0           320 ppi
hdpi：    1.5     
mdpi：   1.0（基准）  160 ppi
ldpi：     0.75

我们来看一张华为荣耀7的参数:

![荣耀7.png](https://upload-images.jianshu.io/upload_images/2934684-a93399c70a6031b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以在android中布局文件中写 10dp。 在运行时会被android系统根据 px = dp * (dpi / 160) 转为具体的 px 。那么对应上面这个荣耀7就是 26.5 px

### 使用dp做屏幕适配会引发的问题

我们先假设有两个设备:

设备1，屏幕宽度为 1080px，480DPI，屏幕总 dp 宽度为 1080 / (480 / 160) = 360dp

设备2，屏幕宽度为 1440px，560DPI，屏幕总 dp 宽度为 1440 / (560 / 160) = 411dp

那么如果对于一个view，我们都写100dp， 那么这个view再被渲染的时候，在设备1上看起来会明显比设备2大。这是因为他们占的宽度比不一样：
(100 / 360 = 0.278)   -> 27%
(100 / 411 = 0.243)   -> 24%

## 今日头条屏幕适配

最近关于屏幕适配在网络上讨论的最多的就是今日头条的方案,而这个方案的核心公式是: 屏幕的宽度总 px / density = 屏幕的宽度总 dp 

>density就是上面的 (dpi / 160)

可以看到上面使用dp适配引发的问题，那么如果对于屏幕分辨率不同的手机，屏幕的宽度总dp总是相同的，那么我们我们直接按照dp来写view的尺寸就不会有问题了。

那么这个方案就是对于不同分辨率的手机，变化density来使屏幕的宽度总dp是不变的。

## 目前我工作中对于view布局方式

目前我并没有采用今日头条这种方案做尺寸适配。一般设计会以 iphone8 : 375 * 667, 来给android和ios出一套图。我直接按照设计的标注，直接在布局文件中使用dp。这样做在Android中做出的效果设计师是可以接受的，那这会有什么问题呢？

我今天仔细想了一下我们为什么会这么做，以及这么做会引发什么问题。 在看这个之前我们先来看一下iphone手机的屏幕尺寸:
![iphone尺寸.png](https://upload-images.jianshu.io/upload_images/2934684-487b4cb3c06e7bcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在ios中有类似于android中dp的单位，pt。

可以看到如果ios的同学如果直接写pt，那么在 iphone8 plus 也会出现android中dp适配所出现的问题，因为屏幕的宽度总dp不一样。

### 使用iphone8 (375*667)会引发的问题

我们还是以荣耀7为例:1920 * 1080, 424ppi, (dpi / 160) = 2.65 。那么转化为dp的屏幕宽高就是 : 724(dp) * 407(dp)。 不论在宽还是高都大于设计师给的 (667 * 375)。

以宽为例，如果我直接按照设置图纸来布局的话，那么宽是会出现不够的情况的:

![diffrentdp.png](https://upload-images.jianshu.io/upload_images/2934684-cbb6101f95941d3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其实这就是上面dp适配的问题。

### 为什么这样做又是ok的呢？

>首先我们来看一下常见的手机的尺寸参数

http://www.woshipm.com/screen/index.html

可以看出大部分的手机的dp宽高参数都是比较靠近 : 667 * 375。 所以在布局的时候只要不要出现在一个方向上所有布局参数都写死就是没太大问题:

比如：

1. 对于375dp的宽度， 3个view的宽你都写了固定值，且相加等于375。那么在不同的手机上由于屏幕总dp宽不同，那么可能就会有空余或者显示不够的情况。
2. 对于375dp的宽度， 2个view的宽写了固定值，剩余那个view占用剩余空间，那么在不同的手机上显示效果还是ok的。

所以设计师在做设计的时候就会考虑这种情况，使得那些 多余或者缺少的空间不会影响显示的效果。不过问题确实是有的。



参考文章：

https://www.jianshu.com/p/c3387bcc4f6e

https://juejin.im/post/5b7a29736fb9a019d53e7ee2










