> View的事件体系

## Point
* View是Android中所有控件的基类，它是一种界面层的控件的一种抽象，代表了一个控件。
* ViewGroup继承自View，它的内部可以包含多个View。


### View的位置
* View的位置参数的四个坐标都是相对于父容器的。

    ```
        width = right - left
        height = bottom - top
    ```   
    
* x和y是View左上角的坐标，translationX和translationY是View左上角相对于父容器的偏移量

    ```
        x = left + translationX
        y = top + translationY
    ```
    
* View在平移的过程中，top和left表示的是原始左上角的位置信息，其值不会发生改变。此时发生改变的x、y、translationX和translationY

### 事件处理中常用的对象
* TouchSlop:它是系统所能识别出的被认为是滑动的最小距离。（在某版本的值为8dp）
    * 可以通过 ： `ViewConfiguration.get(context).getScaledTouchSlop`
* VelocityTracker:用来追踪手指在滑动过程中的速度。
* GestureDetector:用于辅助检测用户的单击、滑动、长按、双击等行为。
* Scroller:用来实现View的弹性滑动，需要和View的computeScroll方法配合使用。

## View的滑动
* 通过View本身提供的scrollTo/scrollBy方法实现滑动
* 通过动画给View施加平移效果实现滑动
* 改变View的LayoutParams使得View重新布局从而实现滑动

### scrollTo和scrollBy
> 这两个方法可以通过改变View内部的两个属性mScrollX和mScrollY来实现View内容的滑动。

- mScrollX的值总是等于View左边缘和View内容左边缘在水平方法的距离。
- mScrollY的值总是等于View上边缘和View内容上边缘在竖直方法的距离。
- 当View左边缘在View内容左边缘，mScrollX为正值，反之为负值。
- 当View上边缘在View内容上边缘的下边时，mScrollY为正值，反之为负值。
- 通过这两个方法，不管怎么滑动，也不可能将当前View滑动到附近View所在区域。


### 使用动画滑动View
> 可以使用View动画或者属性动画完成View的滑动。

- View动画并不能真正改变View的位置
- 属性动画可以改变View的位置，具体通过改变(类似translationX的属性)，属性动画在3.0以下需要使用兼容库。

### 改变布局参数
> 这里有两种思想

- 真正改变View的布局参数
    
    ```
        val params = view.layoutParams;
        params.width += 10
        view.requestLayout()
        //view.layoutParams = params
    ```
    
- 通过其他View来改变自己的位置
    > 比如在一个LinearLayout布局中，我们可以在动画View的左边放一个宽度为0的View，当动画时，只需要重新设置空View的宽度即可
    
    
### View三种滑动方式的对比
- scrollTo/scrollBy : 操作简单，适合对View内容滑动
- 动画 : 操作简单，主要适用于没有交互的View和实现复杂的动画效果
- 改变布局参数 : 适用于有交互的Vie
    



