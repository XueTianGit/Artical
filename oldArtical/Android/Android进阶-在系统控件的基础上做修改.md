title: Android项目-在系统控件的基础上做修改
date: 2016/3/1 9:12:47             
categories: Android
---

# Android项目-在系统控件的基础上做修改 #

在这里，自定义一个ProgressBar（圆形）， 其实这里说是自定义，就是想把那个圆形的样子换成别的样子。

1）首先，Android原生的ProgressBar（圆形）是这样定义的：

    <style name="Widget.ProgressBar">
        <item name="android:indeterminateOnly">true</item>
        <item name="android:indeterminateDrawable">@android:drawable/progress_medium_white</item>
        <item name="android:indeterminateBehavior">repeat</item>
        <item name="android:indeterminateDuration">3500</item>
        <item name="android:minWidth">48dip</item>
        <item name="android:maxWidth">48dip</item>
        <item name="android:minHeight">48dip</item>
        <item name="android:maxHeight">48dip</item>
        <item name="android:mirrorForRtl">false</item>
    </style>****

	可以看到 引用的是这个 -> @android:drawable/progress_medium_white

2）在源码中找到 progress_medium_white， 使用everything搜一下

	打开 progress_medium_white.xml， 可以看到：
	<animated-rotate xmlns:android="http://schemas.android.com/apk/res/android"
	    android:drawable="@drawable/spinner_white_48"
	    android:pivotX="50%"
	    android:pivotY="50%" />

	即是一个转圈的动画。， 引用了 spinner_white_48， 还是使用everything搜一下

3）搜后可以看到他就是系统原生的圆形ProgressBar的图片

4) 因此，我们要改变系统原生的圆形ProgressBar的样式，就把这个图片换了就ok了

5）因此我们可以把progress_medium_white.xml文件，拷贝到我们的项目目录下， 并把图片换成我们想要的

   这样就可以更改系统原生的圆形ProgressBar的样式

	-> ** 主要是因为如果我们把progress_medium_white.xml拷贝到我们的目录下， 那么系统会引用我们的样式！**

6）在我们的 values/styles.xml文件中定义一个样式

    <style name="MyProgressBar" parent="android:Widget.ProgressBar">
        <item name="android:indeterminateDrawable">@drawable/progress_medium_white</item>  //引用我们更换了样式的动画
    </style>

7）在使用ProgressBar的地方，引用这个样式就可以了