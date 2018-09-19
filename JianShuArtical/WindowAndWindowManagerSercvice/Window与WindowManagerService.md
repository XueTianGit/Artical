> Window与WindowManagerService

## Point
* Window表示的是一种抽象的功能集合，具体实现为PhoneWindow，位于WindowManagerServices
* WindowManager是外界访问Window的入口
* WindowManger和WindowManagerService的交互是一个IPC过程
* Android中所有的视图都是通过Window来呈现的，不管是Activity、Dialog、Toast，他们的视图都是附加在Window上的
* Window是一个抽象概念


## 关于对Window与WindowManager

* Window是一个抽象概念，每一个Window都对应着一个View和一个ViewRootImpl，Window和View通过ViewRootImpl来建立联系
* View才是Window存在的实体，可以理解为WindowManager中的addView()方法，即为add一个Window
* 对Window的访问必须通过WindowManager。
    

### 抽象的Window

![Windowg构成](/Users/susion/Documents/susion/Artical/JianShuArtical/WindowAndWindowManagerSercvice/Window'sconsture.png)

上图描述了Window的主要构成。

### Window中View的具体层级

> 下面通过Window的添加过程来看一下Window中View的具体层级：

前面说过，我们对于Window的所有操作都是通过WindowManager来完成的：
WindowManager实现了ViewManager，这个接口定义了我们对于Window的基本操作：

```
public interface ViewManager
{
    public void addView(View view, ViewGroup.LayoutParams params);
    public void updateViewLayout(View view, ViewGroup.LayoutParams params);
    public void removeView(View view);
}
```


> WindowmManager的具体实现为WindowManagerImpl，不过对于具体实现又委托给WindowManagerGlobal。 对于 addView()方法，在实现中主要做3件事：

* 检查参数是否合法，如果是子Window那么还需要调整一些布局参数

* 创建ViewRootImpl并将View添加到列表中

    在WindowManager中有几个用来管理Window的集合：

    ```
     private final ArrayList<View> mViews = new ArrayList<View>();    //存储Window对应的View
     private final ArrayList<ViewRootImpl> mRoots = new ArrayList<ViewRootImpl>();   //Window对应的ViewRootImpl 
     private final ArrayList<WindowManager.LayoutParams> mParams = new ArrayList<WindowManager.LayoutParams>(); //Window对应的布局参数
     private final ArraySet<View> mDyingViews = new ArraySet<View>();  //将要remove的Window对象

     在addView方法中，将会为Window的 View、ViewRootImpl、mParams建立联系：
     
     root = new ViewRootImpl(view.getContext(), display);
     view.setLayoutParams(wparams);
     
     mViews.add(view);
     mRoots.add(root);
     mParams.add(wparams);
  
    
    try {
        root.setView(view, wparams, panelParentView);   //渲染Window
    } catch (RuntimeException e) {
    ```

* 通过ViewRootImpl来更新界面并完成Window的添加过程

    在上面将 View、ViewRootImpl、mParams放到集合中后，会通过ViewRootImpl的setView方法完成View的渲染。
    在这个方法中会调用：

   ```
    /**
     * Called when something has changed which has invalidated the layout of a
     * child of this view parent. This will schedule a layout pass of the view
     * tree.
     */
    public void requestLayout();
   ```

   该方法会完成 View的   onMeasure、onLayout、onDraw

    在setView方法中，会通过Binder对象WindowSession将Window的添加请求发送给WindowManagerService。

![](/Users/susion/Documents/susion/Artical/JianShuArtical/WindowAndWindowManagerSercvice/setview.png)

通过上面的了解，我们对于window的组成有了大致的了解，接下来看一下window的创建过程。

## Window的创建过程

View是Android中的视图的呈现方式，但是View不能单独存在，它必须附着在Window这个抽象概念上，因此有视图的地方就有Window：Activity、Dialog、Toast，他们其实都对应着一个Window。

> 下面以Activity中的Window创建，来分析Window的创建过程:

Activty的实例对象创建后，会通过attach关联activity向关联的上下文对象，并创建相关的Window。Activity实现了Window的Callback，因此当Window接收到外界的状态改变时就会回调Activity的方法，比如：onAttachToWindow、dispatchTouchEvent等。

下面为Activity的attach方法：

``` 
mWindow = new PhoneWindow(this, window);
mWindow.setWindowControllerCallback(this);
mWindow.setCallback(this);
mWindow.setOnWindowDismissedCallback(this);
        mWindow.getLayoutInflater().setPrivateFactory(this);
        if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
                mWindow.setSoftInputMode(info.softInputMode);
        }
        if (info.uiOptions != 0) {
                mWindow.setUiOptions(info.uiOptions);
        }
mUiThread = Thread.currentThread();
```

 从上面可以看出，Window的实现为PhoneWindow。并且该方法完成了Window的创建。

> 在上面我们已经知道，Window的视图展现由View完成的，那么Activity的视图是怎么和View相关联的呢？

首先我们都是在setContentView( )方法中设置Activity的视图：

```
    public void setContentView(int resId) {
        this.ensureSubDecor();
        ViewGroup contentParent = (ViewGroup)this.mSubDecor.findViewById(android.R.id.content);
        contentParent.removeAllViews();
        LayoutInflater.from(this.mContext).inflate(resId, contentParent);
        this.mOriginalWindowCallback.onContentChanged();
    }
```
this.ensureSubDecor(); 这里确保DecorView存在，不存在的话会创建DecorView，

*DecorView是Activity中的顶级View，是一个FrameLayout。一般来说它的内部包含标题栏和内容栏，内容栏的Id为android.R.id.content.他是Activity的Window的View，会被add到WindowManagerService中管理*

可以看到这段代码，就是把我们的布局inflate到DecorView的ContentView中。

> 那和Window有什么联系呢？

其实DecorView来自于Window，从   createSubDecor( ) 方法中可以看出

![](/Users/susion/Documents/susion/Artical/JianShuArtical/WindowAndWindowManagerSercvice/createSubDecor.png)


所以说： Activity的setContentView方法，其实就是把布局设置到了Window的DecorView的contentView中：

> 下图为Activity视图的组成：

![createSubDecor](/Users/susion/Documents/susion/Artical/JianShuArtical/WindowAndWindowManagerSercvice/ActivitycontentView.png)
               
> 而对于DecorView的结构则由系统版本和主题确定。比如没有标题栏等。

*到此为止，Activity的视图已经被inflate到DecorView中，但这个时候DecorView还没有被WindowManager正式添加到Window中，即还没有被管理，
无法提供具体的功能，无法接收外界的输入信息。
在ActivityThread的handleResumeActivity方法中，会首先调用Activity的onResume方法，接着会调用Activity的makeVisible()，在这个方法中，DecorView真正完成了添加和显示。即Activity的视图才能被用户看到。*



### Toast

> Toast也是基于Window实现的，第一类是Toast访问NotificationManagerService，第二类是NotificationManagerService回调Toast里的TN接口。需要注意d的是：

* Toast无法在没有Looper的线程中弹出。
* 这是因为当NMS处理Toast的显示或隐藏请求时会跨进程回调TN中的方法。这里由于TN运行在Binder线程池中，所有需要通过Handler将其切换到当前线程中
* 我们一般是Make一个Toast。插入到NMS的Toast显示队列中，一般对于一个应用，最多有50个。    
* 如果我们想始终显示最新的消息，那么单一Toast对象可以做到这点。
* Toast的样子可以自定义




