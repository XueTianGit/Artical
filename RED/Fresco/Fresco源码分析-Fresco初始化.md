---
title: Fresco设计的基本概述
date: 2018/3/20
tags: 
  - Fresco
  - susion
---

> 本文为分析`Fresco`源码第一篇，基于`fresco 1.8.1`。

### ImagePipeline

> image pipeline 主要负责图片的加载和缓存工作。 可以从网路、本地内存、app res中加载图片。它具有三层缓存，其中两个在内存中，另一个在手机本地存储中。

### Drawees

> Drawees主要负责图片的展示工作。在图片完全加载之前他可以显示一个占位图，当图片不在屏幕上显示时，它可以自动释放图片所占用的内存。

## 从Fresco初始化看Fresco的基本组件

> Fresco.java

```
public static void initialize(
    Context context,
    @Nullable ImagePipelineConfig imagePipelineConfig,
    @Nullable DraweeConfig draweeConfig) {
    //...
    if (imagePipelineConfig == null) {
        ImagePipelineFactory.initialize(context);
    } else {
        ImagePipelineFactory.initialize(imagePipelineConfig);
    }
    initializeDrawee(context, draweeConfig);
}

/** 初始化Drawee. */
private static void initializeDrawee(Context context,@Nullable DraweeConfig draweeConfig) {
    sDraweeControllerBuilderSupplier =
        new PipelineDraweeControllerBuilderSupplier(context, draweeConfig);
    SimpleDraweeView.initialize(sDraweeControllerBuilderSupplier);
}
```

从Fresco的初始化可以看出Fresco的基本组件为：`ImagePipeline`和`Drawee`,接下来仔细看一下这两个都是负责哪些事情:

### ImagePipeline

> 由于`ImagePipeLine`是一个功能十分多的组件，它由`ImagePipelineFactory`负责创建，由`ImagePipelineConfig`负责配置。`ImagePipelineFactory`提供一些`ImagePipeline`组成的组件，比如缓存。
`ImagePipelineConfig`主要负责配置`ImagePipeline`的网络相关选项和缓存相关选项等等。

>ImagePipelineFactory.java

```
    public ImagePipeline getImagePipeline() {
        if (mImagePipeline == null) {
        mImagePipeline =
            new ImagePipeline(
                getProducerSequenceFactory(),
                mConfig.getRequestListeners(),
                mConfig.getIsPrefetchEnabledSupplier(),
                getBitmapMemoryCache(),
                getEncodedMemoryCache(),
                getMainBufferedDiskCache(),
                getSmallImageBufferedDiskCache(),
                mConfig.getCacheKeyFactory(),
                mThreadHandoffProducerQueue,
                Suppliers.of(false));
        }
        return mImagePipeline;
    }
```

>ImagePipelineConfig.java

```
    public class ImagePipelineConfig {
        //...
        private final NetworkFetcher mNetworkFetcher;
        private final int mHttpNetworkTimeout;
        @Nullable private final PlatformBitmapFactory mPlatformBitmapFactory;
        private final PoolFactory mPoolFactory;
        private final ProgressiveJpegConfig mProgressiveJpegConfig;
        private final Set<RequestListener> mRequestListeners;
        private final boolean mResizeAndRotateEnabledForNetwork;
        private final DiskCacheConfig mSmallImageDiskCacheConfig;
        @Nullable private final ImageDecoderConfig mImageDecoderConfig;
        private final ImagePipelineExperiments mImagePipelineExperiments;
        //...
    }
```


### Drawee

在上面的`initializeDrawee()`方法中可以看到，在初始化`SimpleDraweeView`之前，先初始化了`PipelineDraweeControllerBuilderSupplier`,那么这个类是干什么的？

`PipelineDraweeControllerBuilderSupplier`很简单，就是帮助`PipelineDraweeControllerBuilder`提供了`ImagePipeline`和`PipelineDraweeControllerFactory`。顾名思义, `PipelineDraweeControllerFactory`是用来生产`PipelineDraweeController`的。看源码可以知道，`ImagePipeline`是为方便`PipelineDraweeController`获得缓存。

> PipelineDraweeControllerBuilder.java

```
 @Override
  protected PipelineDraweeController obtainController() {
    DraweeController oldController = getOldController();
    PipelineDraweeController controller;
    if (oldController instanceof PipelineDraweeController) {
      controller = (PipelineDraweeController) oldController;
      controller.initialize(
          obtainDataSourceSupplier(),
          generateUniqueControllerId(),
          getCacheKey(),
          getCallerContext(),
          mCustomDrawableFactories,
          mImageOriginListener);
    } else {
      controller =
          mPipelineDraweeControllerFactory.newController(
              obtainDataSourceSupplier(),
              generateUniqueControllerId(),
              getCacheKey(),
              getCallerContext(),
              mCustomDrawableFactories,
              mImageOriginListener);
    }
    return controller;
  }

  //CacheKey 由 mImagePipeline.getCacheKeyFactory()。提供
```

其实上面层层构造，最后是为了产生一个十分关键的对象`PipelineDraweeController`。

### PipelineDraweeController

先来看一下官方注释：

```
Drawee controller that bridges the image pipeline with {@link SettableDraweeHierarchy}. <p> The
hierarchy's actual image is set to the image(s) obtained by the provided data source。
```

大致就是 `PipelineDraweeController`是`ImagePipeline` 和`Drawee`在图片处理流程中的结合点。即把`ImagePipeline`获取的图片数据传递给`Drawee`。

从这个类的属性也可以看出它所承担的工作:

>PipelineDraweeController.java

```
//Drawee相关
private final Resources mResources;
private final DrawableFactory mAnimatedDrawableFactory;
@Nullable
private final ImmutableList<DrawableFactory> mGlobalDrawableFactories;
//数据缓存相关
private @Nullable MemoryCache<CacheKey, CloseableImage> mMemoryCache;
private CacheKey mCacheKey;
// Constant state (non-final because controllers can be reused)
private Supplier<DataSource<CloseableReference<CloseableImage>>> mDataSourceSupplier;

```

不过上面官方注释指出的是图片数据传递给了`SettableDraweeHierarchy`，而并没有直接传递给`Drawee`。那先看一眼`SettableDraweeHierarchy`这个类大概是干什么吧。

>SettableDraweeHierarchy.java

```
Interface that represents a settable Drawee hierarchy. Hierarchy should display a placeholder
image until the actual image is set. In case of a failure, hierarchy can choose to display
a failure image.
```

恩，明白，大致是控制加载不同阶段图片显示的切换的

**继续来看Drawee的初始化:**

初始化所做的工作十分简单:

>SimpleDraweeView.java

```
 private void init(Context context, @Nullable AttributeSet attrs) {
    if (isInEditMode()) {
      return;
    }
    Preconditions.checkNotNull(
        sDraweeControllerBuilderSupplier,
        "SimpleDraweeView was not initialized!");
    mSimpleDraweeControllerBuilder = sDraweeControllerBuilderSupplier.get();

    if (attrs != null) {
      TypedArray gdhAttrs = context.obtainStyledAttributes(
          attrs,
          R.styleable.SimpleDraweeView);
      try {
        if (gdhAttrs.hasValue(R.styleable.SimpleDraweeView_actualImageUri)) {
          setImageURI(
              Uri.parse(gdhAttrs.getString(R.styleable.SimpleDraweeView_actualImageUri)),
              null);
        } else if (gdhAttrs.hasValue((R.styleable.SimpleDraweeView_actualImageResource))) {
          int resId = gdhAttrs.getResourceId(
              R.styleable.SimpleDraweeView_actualImageResource,
              NO_ID);
          if (resId != NO_ID) {
            setActualImageResource(resId);
          }
        }
      } finally {
        gdhAttrs.recycle();
      }
    }
  }
```

判断`SimpleDraweeControllerBuilder`是否存在，不存在则初始化失败。如果设置`SimpleDraweeView`的属性，则对属性做相应的处理。

额。上面我们看了`PipelineDraweeController`。那`SimpleDraweeController`又是什么呢？

### SimpleDraweeController

其实`SimpleDraweeController`就是`PipelineDraweeController`。这点可以从类的继承层次可以看出：

```
PipelineDraweeControllerBuilderSupplier implements Supplier<PipelineDraweeControllerBuilder> 

PipelineDraweeControllerBuilder extends AbstractDraweeControllerBuilder<PipelineDraweeControllerBuilder

 AbstractDraweeControllerBuilder <
    BUILDER extends AbstractDraweeControllerBuilder<BUILDER, REQUEST, IMAGE, INFO>,
    REQUEST,
    IMAGE,
    INFO>
    implements SimpleDraweeControllerBuilder
```

>上面对于Fresco的初始化大致已经分析完成了，对于Fresco图片加载的框架也基本比较清晰了，大概可以用下面这张图描述：

![](http://7xrbxa.com1.z0.glb.clouddn.com/galaxywarshipFrescoBasicArticture.png)


>可能存在一些问题，待后续完整阅读后，会慢慢更正。