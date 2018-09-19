## View的工作原理
### ViewRoot和DecorView

ViewRoot对应于ViewRootImpl类，它是连接WindowManager和DecorView的纽带，View的三大流程均是通过ViewRoot来完成的。

>在ActivityThread中，当Activity对象被创建完毕后，会将DecorView添加到Window中，同时会创建ViewRootImpl对象，并将ViewRootImpl对象和DeocorView关联。

```
    root = new ViewRootImpl(view.getContext(), display)
    root.setView(view, wparams, panelParentView)
```

### onMeasure、onLayout、onDraw
View的绘制流程是从ViewRoot的performTraversals方法开始的。并依次经历measure、layout、draw。

- measure决定了View的宽高，Measure完成后，可以通过`getMeasuredWidth`和`getMeasuredHeight`方法来获取到View测量后的宽高，在几乎所有的情况下它都等同于View最终的宽高

- layout决定了View的四个顶点的坐标和实际的View的宽高。完成后，可以通过`getTop getBottom getLeft getRight`来拿到View的四个顶点的位置，并可以通过`getWidth getHeight`来拿到View的最终宽和高

- draw决定了View的显示，只有draw方法完成以后View的内容才能呈现在屏幕上。


### View的测量
View的宽高由MeasureSpec决定。对于普通View，其MeasureSpec由父容器的MeasureSpec和自身的LayoutParams来共同决定。

>特殊的对于DecorView，其MeasureSpec由窗口的尺寸和其自身的LayoutPrams来共同决定。
 
#### MeasureSpec
>MeasureSpec代表一个32位的int值，高2位代表SpecMode，低30位代表SpecSize，SpecMode指测量模式，而SpecSize是指在某种测量模式下的规格大小。

SpecMode有3类

- UNSPECIFIED:父容器不对View有任何限制，要多大给多大。

- EXACTLY:父容器已经检测出View所需要的精确大小，这个时候View的最终大小就是SpecSize所指定的值。它对应LayoutParams中的math_parent和具体数值这两种模式。

- AT_MOST:父容器指定了y一个可用大小即SpecSize，View的大小不能大于这个值，具体是什么要看不同View的具体实现。它对应于LayoutParams中的wrap_content。


从 measureChildWithMargins()方法可用看出：
    即根据父容器的MeasureSpec同时结合View本身的LayoutParams来确定子元素的MeasureSpec

```
    final int childWidthMeasureSpec = 
    getChildMeasureSpec(parentWidthSMeasureSpec, 
    parentPaddingLeft+parentPadingRight+childLp.leftMargin+childLp.rightMargin+widthUesed,
    childLp.width)   
```

简单说一下 getChildMeasureSpec() 这个方法的实现:
  当View采用固定宽/高的时候，不管父容器的MeasureSpec是什么，View的MeasureSpec都是精确模式并且其大小遵循LayoutParams中的大小；
  当View的宽/高是match_parent时，如果父容器是精确模式，那么View也是精确模式并其其大小是父容器的剩余空间，如果父容器是最大模式，那么View也是最大模式并且其大小不会超过父容器的剩余空间；
  当View的宽/高是wrap_content时，不管父容器的模式是精准还是最大化，View的模式总是最大化并且大小不能超过父容器的剩余空间。
  
##### measure()
measure是一个final方法，最终会调用onMeasure()，看一下View的onMeasure方法的实现：

```
setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth()), widthMeasureSpec)
,getDefaultSize(getSuggestedMinimumHeight()), heightMeasureSpec)
```

对于getDefaultSize方法：
在AT_MOST和EXACTLY时，返回的大小就是 widthMeasureSpec中的size，即这个size就是View测量后的大小。
在UNSPECIFIED时，以宽为例，如果View没有设置背景，则View的宽度即为android:minWidth。如果没指定则为0。如果View设置了背景，则去android:minWidth和背景的最小宽度这两者中的最大值。

**测量小结**
>结合measureChildWithMargins()和onMeasure()的实现可以得到如下结论：直接继承View的自定义控件需要重写onMeasure方法并设置wrap_content时自身大小。否则在布局中 使用wrap_content就相当于使用math_parent。对于这个问题，我们可以这样解决：在View使用wrap_content时，我们可以指定一个给它指定一个默认的宽高。

```
    setMeasuredDimension(mDefaultWidth, mDefaultHeight)
```
>在onMeasure中拿到的测量宽高很可能是不准确的，一个比较好的习惯是在onLayout方法中去获取View的测量宽/高或者最终的宽高。

**获取宽高**
>有时我们想再Activity的生命周期中获取View的宽高，但是Activity的生命周期和View的measure过程并不是同步的，这时候我们可以通过以下方法确保获得准确的View宽高：

```
1. View.post(runnable) 在Looper调用此runnable时，View已经初始化完毕。
2. ViewTreeObserver的onGlobalLayoutListener， 这个接口会在View树的状态发生改变或者View树内部的View的可见性发生改变时调用。但需要注意的是这个方法会被调用多次，如果你对View的宽高有实时性的要求，这个方法不错。
3. view.measure(widthMeasureSpec,heightMeasureSpec) 手动测量
    这里根据View的LayoutParams分情况看一下参数如何传递：
    
    math_parent:
    无法measure出具体的宽高,这是因为构造此种MeasureSpec需要知道parentSize，
    即父容器剩余空间大小，但这时是无法知道的。
    
    具体数值:
    widthSpec = MeasureSpec.makeMeasureSpec(100, MeasureSpec.EXACTLY)
    view.measure(widthSpec, heightSpec)
    
    wrap_content:
    widthSpec = MeasureSpec.makeMeasureSpec((1<<30) -1, MeasureSpec.AT_MOST)
    view.measure(widthSpec, heightSpec)
    即假设剩余可使用的空间为最大值，在这个值的情况下去测量View的大小。
```

### View的Layout
layout方法确定View本身的位置。onLayout方法则会确定所有子元素的位置。

Layout方法首先会通过setFrame方法来设定View的四个顶点的位置，即初始化mLeft，mRight，mTop和mBottom这四个值，View的四个顶点一旦确定，那么View在父容器中的位置也就确定了。

对于ViewGroup的onLayout方法，已LinearLayout为例，在他的layoutVertical中，此方法会遍历所有子元素并调用setChildFrame方法来为子元素指定对应的位置，其childTop会逐渐增大，即后面的子元素会被放置在靠下的位置。 setChildFrame的实现仅仅是调用子元素的layout方法而已。

View的getMeasureWidth和getWidth方法是没有区别的：

```
    childWidth = child.getMeasuredWidth()
    setChildFrame(child, childLeft,childTop+getLocationOffset(child), childWidth,
    childHeight)
    
    setChildFrame的具体实现：
        child.layout(left, top, left + width, top + height)
        
        mRight = left + width; mLeft = left 
        
    getWidth的实现：
        return mRight - mLeft
```

那么在那些特殊情况下会不同呢？

    ```
        如果重写layout方法：
        
        public void layout(int l, int t, int r, int b){
            super.layout(l, t, r+100, b+100)
        }
        
        即故意使View的宽、高各变大100。
        
        另外一种情况是：
        
        在View需要多次measure才能确定自己测量宽高，在前几次的测量过程中，
        其得出的测量宽高可能和最终宽高不一致，但最终来说，测量宽高还是和最终宽高相同。
    ```
    
    
### View的draw过程
>View的绘制过程分为如下几步：
1. 绘制背景`background.draw(canvas)` 
2. 绘制自己`onDraw()`
3. 绘制children `dispatchDraw()`
4. 绘制装饰 `onDrawScrollBars`

在View的绘制过程中有一个标记为`setWillNotDraw`，如果设置了这个标记位，系统会认为这个View不会绘制任何内容，会进行相应的优化。View不会启用这个标记为，但是ViewGroup会默认启用这个标记位，因此，当我们自定义的ViewGroup需要通过onDraw来绘制内容时，我们需要显示地关闭这个标记位。





  
  






