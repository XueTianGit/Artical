---
title: 图片加载初探
date: 2018/4/3
tags: 
  - Fresco
  - susion
---

> 在使用Fresco时，最常用的莫过于这个方法:`void setImageURI(@Nullable String uriString)`。那么就以这个方法为起点，追踪fresco的图片加载流程。

## setImageURI()

>SimpleDraweeView.java

```
  public void setImageURIroller = mSimpleDraweeControllerBuilder
        .setCallerContext(callerContext)
        .setUri(uri)
        .setOldController(getController())
        .build();
    setController(controller);
  }
```
可以看到这个方法就设置了一个`DraweeController`。其实就是一个`PipelineDraweeController`对象。这个对象持有要加载图片的Uri，那么看来对于图片的加载`PipelineDraweeController`是关键。

> 继续追踪 `setController` -> `DraweeView.java`

```
  public void setController(@Nullable DraweeController draweeController) {
    mDraweeHolder.setController(draweeController);
    super.setImageDrawable(mDraweeHolder.getTopLevelDrawable());
  }
```

即实际上是把Controller设置给了`DraweeHolder`。并立刻显示`DraweeHolder`中的最顶层的图片。

>第一章已经知道Fresco在加载图片时，对于不同的阶段会显示不同的图片，具体使用一个`DraweeHierarchy`的概念来组织。这里其实可以猜出`mDraweeHolder.getTopLevelDrawable()`就是拿出`DraweeHierarchy`中我们设置的 placeholder 图片。

这段代码把要做的事情都委托给了`DraweeHolder`。在看具体下面流程之前，先大致看一下`DraweeHolder`是干什么的。

### DraweeHolder

```
/**
 * A holder class for Drawee controller and hierarchy.
 *
 * <p>Drawee users, should, as a rule, use {@link DraweeView} or its subclasses. There are
 * situations where custom views are required, however, and this class is for those circumstances.
 *
 * <p>Each {@link DraweeHierarchy} object should be contained in a single instance of this
 * class.
   ...
 */
public class DraweeHolder<DH extends DraweeHierarchy> implements VisibilityCallback 
```

即这个类看样是维护 Drawee controller 和 hierarchy关系的。`DraweeHierarchy`和`DraweeHolder`是一对一的关系。

>接下来继续追踪 `setController` -> `DraweeHolder`

```
  public void setController(@Nullable DraweeController draweeController) {
    boolean wasAttached = mIsControllerAttached;
    if (wasAttached) {
      detachController();
    }

    // Clear the old controller
    if (isControllerValid()) {
      mEventTracker.recordEvent(Event.ON_CLEAR_OLD_CONTROLLER);
      mController.setHierarchy(null);
    }
    mController = draweeController;
    if (mController != null) {
      mEventTracker.recordEvent(Event.ON_SET_CONTROLLER);
      mController.setHierarchy(mHierarchy);
    } else {
      mEventTracker.recordEvent(Event.ON_CLEAR_CONTROLLER);
    }

    if (wasAttached) {
      attachController();
    }
  }
```

上面代码维护了Hierarchy与Controller的关系，一个Hierarchy被一个Controller引用。把Controller操作相关的一些事件记录到`DraweeEventTracker`。然后调用`attachController()`。

>DraweeEventTracker是一个`This class keeps a record of internal events that take place in the Drawee`。他主要用来调试。

`attachController()`最终会调用 `mController.onAttach()`，在这个方法中发起了图片加载事件。那么来看一下`onAttach()` -> `AbstractDraweeController.java`

```
  public void onAttach() {
    mEventTracker.recordEvent(Event.ON_ATTACH_CONTROLLER);
    Preconditions.checkNotNull(mSettableDraweeHierarchy);
    mDeferredReleaser.cancelDeferredRelease(this);
    mIsAttached = true;
    if (!mIsRequestSubmitted) {
      submitRequest();
    }
  }
```
可以看到这个方法主要记录了一些状态，并且要`mSettableDraweeHierarchy`不为null,这里其实就有点问题了，我们其实并没有在加载图片时设置过`DraweeHierarchy`那这个是哪里来的呢？其实我们在布局文件，或者手动new `SimpleDraweeView`时都会触发`GenericDraweeView`的`inflateHierarchy`。触发时机当然是在设置`DraweeHierarchy`时。

```
  protected void inflateHierarchy(Context context, @Nullable AttributeSet attrs) {
    GenericDraweeHierarchyBuilder builder =
        GenericDraweeHierarchyInflater.inflateBuilder(context, attrs);
    setAspectRatio(builder.getDesiredAspectRatio());
    setHierarchy(builder.build());
  }
```

即`mSettableDraweeHierarchy`其实就是一个`GenericDraweeHierarchy`对象。在判断这个对象不为null后，Controller调用了`submitRequest()`,这个方法看起来好像开始加载图片了。

```
  protected void submitRequest() {
    final T closeableImage = getCachedImage();
    if (closeableImage != null) {
      mDataSource = null;
      mIsRequestSubmitted = true;
      mHasFetchFailed = false;
      mEventTracker.recordEvent(Event.ON_SUBMIT_CACHE_HIT);
      getControllerListener().onSubmit(mId, mCallerContext);
      onImageLoadedFromCacheImmediately(mId, closeableImage);
      onNewResultInternal(mId, mDataSource, closeableImage, 1.0f, true, true);
      return;
    }
    ......
    final String id = mId;
    final boolean wasImmediate = mDataSource.hasResult();
    final DataSubscriber<T> dataSubscriber =
        new BaseDataSubscriber<T>() {
          @Override
          public void onNewResultImpl(DataSource<T> dataSource) {
            boolean isFinished = dataSource.isFinished();
            float progress = dataSource.getProgress();
            T image = dataSource.getResult();
            if (image != null) {
              onNewResultInternal(id, dataSource, image, progress, isFinished, wasImmediate);
            } else if (isFinished) {
              onFailureInternal(id, dataSource, new NullPointerException(), /* isFinished */ true);
            }
          }
          @Override
          public void onFailureImpl(DataSource<T> dataSource) {
            onFailureInternal(id, dataSource, dataSource.getFailureCause(), /* isFinished */ true);
          }
          @Override
          public void onProgressUpdate(DataSource<T> dataSource) {
            boolean isFinished = dataSource.isFinished();
            float progress = dataSource.getProgress();
            onProgressUpdateInternal(id, dataSource, progress, isFinished);
          }
        };
    mDataSource.subscribe(dataSubscriber, mUiThreadImmediateExecutor);
  }
```

大体上逻辑是这样的:**在有缓存的情况下直接使用缓存中的图片，在没有缓存时，使用DataSource加载图片。**

看一下是如何从内存中获取缓存图片的：`PipelineDraweeController -> getCachedImage()`:

```
 protected CloseableReference<CloseableImage> getCachedImage() {
    if (mMemoryCache == null || mCacheKey == null) {
      return null;
    }
    // We get the CacheKey
    CloseableReference<CloseableImage> closeableImage = mMemoryCache.get(mCacheKey);
    if (closeableImage != null && !closeableImage.get().getQualityInfo().isOfFullQuality()) {
      closeableImage.close();
      return null;
    }
    return closeableImage;
  }
```

`CloseableReference`实际上是一个持有引用计数的指针，当他所引用的对象引用数量为0，调用`close()`。

 `private @Nullable MemoryCache<CacheKey, CloseableImage> mMemoryCache`

那么如何使用DataSource加载图片的呢?

```
  final boolean wasImmediate = mDataSource.hasResult();
```

## DataSource

跟着`mDataSource.hasResult()`,追踪一下mDataSource是哪个类的实例,如何加载数据。

`dataSoure`通过 `AbstractDraweeControllerBuilder -> obtainDataSourceSupplier()`方法获得。

```
  /** Gets the top-level data source supplier to be used by a controller. */
  protected Supplier<DataSource<IMAGE>> obtainDataSourceSupplier() {
    //....
    // final image supplier;
    if (mImageRequest != null) {
      supplier = getDataSourceSupplierForRequest(mImageRequest);
    ....
    return supplier;
  }
  ```

可以看到应根据不同的image request获得不同的datasource。追踪源码，最终的data soure是在`ImagePipeline`中`fetchDecodedImage()`方法获得。

```
  public DataSource<CloseableReference<CloseableImage>> fetchDecodedImage(
      ImageRequest imageRequest,
      Object callerContext,
      ImageRequest.RequestLevel lowestPermittedRequestLevelOnSubmit) {
    try {
      Producer<CloseableReference<CloseableImage>> producerSequence =
          mProducerSequenceFactory.getDecodedImageProducerSequence(imageRequest);
      return submitFetchRequest(
          producerSequence,
          imageRequest,
          lowestPermittedRequestLevelOnSubmit,
          callerContext);
    } catch (Exception exception) {
      return DataSources.immediateFailedDataSource(exception);
    }
  }

  private <T> DataSource<CloseableReference<T>> submitFetchRequest(
      Producer<CloseableReference<T>> producerSequence,
      ImageRequest imageRequest,
      ImageRequest.RequestLevel lowestPermittedRequestLevelOnSubmit,
      Object callerContext) {
      ......
      return CloseableProducerToDataSourceAdapter.create(
          producerSequence,
          settableProducerContext,
          requestListener);
    } catch (Exception exception) {
      return DataSources.immediateFailedDataSource(exception);
    }
  }
  ...
```

可以看到这里DataSource返回的是`CloseableProducerToDataSourceAdapter`的实例。那么数据源到底如何发起图片加载请求的呢？看一下`CloseableProducerToDataSourceAdapter`构造函数：

```
  protected AbstractProducerToDataSourceAdapter(
      Producer<T> producer,
      SettableProducerContext settableProducerContext,
      RequestListener requestListener) {
      //....
     producer.produceResults(createConsumer(), settableProducerContext);
  }
```

### 不同的Sequence

上面说不同的imageReuqest会获得不同的DataSource。其实在Fresco中大部分获取获取的Datadource都是来自`fetchDecodedImage`方法。不过这个方法会去获取图片处理的Producer,Producer是根据不同的uri来获取的。



这里其实就是调用`producer`的`produceResults()`来产生请求结果，并通过回调来设置返回结果。而上面的`mDataSource.hasResult()`就是判断当前有没有返回结果。

通过上面的源码分析，这里已经知道，`DataSource`并不会去加载图片，它会把图片加载的细节委托给`Producer`,它只是负责发起`Producer`的图片加载请求，并监听返回结果给外部。

使用下面这张图总结一下流程：

 ![](http://7xrbxa.com1.z0.glb.clouddn.com/galaxywarshipprogress.png)
















