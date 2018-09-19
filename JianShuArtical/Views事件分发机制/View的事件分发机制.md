>View的事件分发机制

##事件的传递规则
>先来看一些定义

* 事件分发其实就是对MotionEvent事件的分发。
* 一个事件序列：从手指接触屏幕的那一刻起，到手指离开屏幕的那一刻结束，在这个过程中所产生的一系列事件，这个事件序列以down事件开始，中间含有数量不定的move事件，最终以up事件结束。
* 一个事件只能被一个View拦截且消耗。

> 对于一个事件序列的分发是由下面三个方法共同完成的

    dispatchTouchEvent(ev:MotionEvent):Boolean

>如果事件能够传递给当前View，那么此方法一定会被调用，返回结果受当前View的onTouchEvet和下级View的dispatchTouchEvent方法的影响，表示是否消耗当前事件。

    onInterceptTouchEvent(ev:MotionEvent):Boolean

>在上面方法内部调用，用来判断是否拦截事件，如果当前View拦截了某个事件，那么在同一个事件序列中，此方法不会再次被调用。

    onTouchEvent(ev:MotionEvent):Boolean

>在dispatchTouchEvent内调用，返回结果表示是否消耗当前事件，如果不消耗，则在同一个事件序列中，当前View无法再次接收到当前事件

>上述三个方法的调用逻辑伪代码如下：

```
fun dispatchTouchEvent(ev:MotionEvent):Boolean{
    var consume = false
    if(onInterceptTouchEvent(ev)){
        consume = onTouchEvent(ev)
    }else{
        consume = child.dispatchTouchEvent(ev)
    } 
    return consume
}
```

##事件分发的具体过程
>首先看下图：

![](/Users/susion/Documents/susion/Artical/JianShuArtical/Views事件分发机制/dispatchEvent.png
)

> 从上图可以看出大概的流程，不过这里有一点需要说一下：

看一下下面这段代码：

```
    if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null){
        final boolean disallowIntercept = (mGroupsFlags & FLAG_DISALLOW_INTERCEPT) != 0;
        
        if(!disallowIntercept){
            intercepted = onInterceptTouchEvent(ev)
            ev.setAction(action);
        } else {
	          intercepted = false;
        }
    } else {
        intercepted = true;
    }
```

> 从上图和代码可以得知：

- 对于ACTIONDOWN事件，可以认为这个事件的分发过程，是为后续事件寻找处理者的过程。
    - 找到的处理者，会被赋值给 mFirstTouchTarget
- FLAG_DISALLOW_INTERCEPT一旦被设置，ViewGroup将无法拦截除了ACTION_DOWN以外的其他点击事件
    - 这个标记位一般由子View通过调用requestDisallowInterceptTouchEvent来设置。
    - ViewGroup在分发事件时，如果是ACTION_DOWN就会重置FLAG_DISALLOW_INTERCEPT这个标记位，从而，对于ACTION_DOWN事件，ViewGroup总是会调用自己的onInterceptTouchEvent方法询问自己是否要拦截事件。
- 当ViewGroup决定拦截事件后，那么后续的点击事件将会默认交给它处理并且不会再调用的onInterceptTouchEvent方法。

### 如何将事件分发给子View
>首先遍历所有子元素，如果满足下面两个条件，则将事件交给它处理。即调用dispatchTouchEvent

    1. 子元素是否在播放动画
    2. 点击事件的坐标是否落在子元素的区域内

> 如果dispatchTouchEvent返回true，则mFirstTouchTarget将会被赋值，同时终止对子元素的遍历。

###View对事件进行处理
>如果将事件分发到了View，由于View没有onInterceptTouchEvent方法，因此事件处理就比较简单。

    1. 首先会判断有没有设置OnTouchListener,并调用listener，如果返回true，则事件被消耗，onTouchEvent将不会被调用。

>对于OnTouchEvent

    1.只要View的CLICKABLE和LONGCLICKABLE有一个为true，那么就会消耗这个事件。即使这个控件为Disable状态
    2.在View出于不可用的状态下，依然会消耗点击事件。即返回true
    3.如果View设有TouchDelegate，那么会调用其onTouchEvent方法。
    4.在ACTION_UP事件发生时，会触发performClick方法，如果View设置了OnClickListener，则执行onClick方法。
    5.setOnClickListener会自动将View的CLICKABLE设置为true。


