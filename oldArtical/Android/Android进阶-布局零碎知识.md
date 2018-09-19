title: Android进阶-布局零碎知识
date: 2016/3/7 18:26:07    
categories: Android
---

# Android进阶-布局零碎知识 #

-
>- 不要老是想着findViewById()这个方法， 在很多应用中，我们可能使用View.inflate()也不少
>- RadioButton + RadioGroup 多个选项的单一选择，并且还很好处理
>    - RadioGroup实际是继承自线性布局
>    - 必须给RadioButton分配ID，这样RadioGroup才可以管理（事件监听什么的）
>- 扩大一个控件，在其父控件中的范围，我们可以使用padding属性来搞定
>- 为了使图片在页面中展示的更加合理
>    - scaleType="centerCrop"  //指定图片从中部裁剪
>    - 其他属性还有很多，比如 fitXY等（不过图片会变形）
>- 图片的宽高，由于屏幕适配的问题， 有时都是写死的
>- ListView
>    - 避免ListView在滑动时出现黑色背景： cacheColorHint="#fff" //弄成白色
>    - 一个ListView是可以添加多个Header和Footer的， 这些Header与Footer会在getPosition()中，占位置的
>    - 对ListView的某个条目进行重绘的原理是：在onItemClick()中有一个View参数， 这个参数就是当前点击的条目，可以直接对它操作
>- 有时候直接对根部局加一些属性，可能并没有达到你预期的效果，此时，可以在根部局下套一个和根布局一样的布局，让后把想要加的属性加在上面
>- 在<shape/>上是可以套一个<rotate/>的，即我们可以直接制作出带动画的简单图形
>- 在代码中，直接弄多选dialog时，点击事件中的which参数是不能用的， 它永远都是指向第一项