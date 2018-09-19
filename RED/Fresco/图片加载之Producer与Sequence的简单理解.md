---
title: 图片加载之Producer与Sequence的简单理解
date: 2018/4/11
tags: 
  - Fresco
  - susion
---

>上一节阅读到：`DataSource`并不会去加载图片，它会把图片加载的细节委托给`Producer`,它只是负责发起`Producer`的图片加载请求，并监听返回结果给外部。

> 首先小小理解一下Fresco中的Producer的设计思想

## Producer

- 接口定义

```
/**
 * Building block for image processing in the image pipeline.
 *
 * <p> Execution of image request consists of multiple different tasks such as network fetch,
 * disk caching, memory caching, decoding, applying transformations etc. Producer<T> represents
 * single task whose result is an instance of T. Breaking entire request into sequence of
 * Producers allows us to construct different requests while reusing the same blocks.
 */
public interface Producer<T> {
  /**
   * Start producing results for given context. Provided consumer is notified whenever progress is
   * made (new value is ready or error occurs).
   * @param consumer
   * @param context
   */
  void produceResults(Consumer<T> consumer, ProducerContext context);
}

```

一个Producer用来处理整个Fresco图片处理流程中的一步，比如网络获取图片、磁盘获取、内存获取等等。我们可以通过组装不同的producer来对整个图片加载流程做不同的处理。
对于每一步通过调用`produceResults`开始。`ProducerContext`可以理解为图片加载请求中的上下文对象，通过它我们可以获得图片的uri、监听器等。`Consumer`担当整个处理流程过程中顶级回调的监听的角色。

- Producer串联图片处理流程

这里以 `BitmapMemoryCacheProducer`为例：对于许多Producer而已，它都有一个`mInputProducer`。这个producer是用在，如果本producer没有完成图片处理流程，那么会把producer处理传给`inputProducer`让它拿着本次处理的结果继续处理：

```
public BitmapMemoryCacheProducer(
      MemoryCache<CacheKey, CloseableImage> memoryCache,
      CacheKeyFactory cacheKeyFactory,
      Producer<CloseableReference<CloseableImage>> inputProducer) {
    mMemoryCache = memoryCache;
    mCacheKeyFactory = cacheKeyFactory;
    mInputProducer = inputProducer;
  }

   @Override
  public void produceResults(
    //... BitmapMemoryCacheProducer处理流程，完成则返回。 return

    mInputProducer.produceResults(wrappedConsumer, producerContext);
  }

```

对于`wrappedConsumer`，上面我们已经知道 `Consumer` 起到一个流程监听,而这里对于`Consumer`又封装了一层，即加了一层代理。

```
protected Consumer<CloseableReference<CloseableImage>> wrapConsumer(
      final Consumer<CloseableReference<CloseableImage>> consumer,
      final CacheKey cacheKey) {
    return new DelegatingConsumer<
        CloseableReference<CloseableImage>,
        CloseableReference<CloseableImage>>(consumer) {

}
```

对于为什么要对`Consumer`加这一层代理，我想是为了本`producer`可以监听到最终结果，比如`BitmapMemoryCacheProducer`它需要得到最终的图片，并对图片做缓存处理。看一下 `BitmapMemoryCacheProducer`的代理`Consumer`在监听到图片请求结果后是如何处理的。

```
  @Override
      public void onNewResultImpl(
          CloseableReference<CloseableImage> newResult,
          @Status int status) {
        //....
        // cache and forward the new result
        CloseableReference<CloseableImage> newCachedResult =
            mMemoryCache.cache(cacheKey, newResult);
        try {
          if (isLast) {
            getConsumer().onProgressUpdate(1f);
          }
          getConsumer().onNewResult(
              (newCachedResult != null) ? newCachedResult : newResult, status);
        } 
        //...
      }
    };
  ```

即：1. 缓存图片请求结果 。 2. 向wrapper的consumer继续传递处理结果。

## Sequence

>继续分析`DataSource`是如果使用`Producer`发起图片加载的

```
  final DataSubscriber<T> dataSubscriber =
        new BaseDataSubscriber<T>() {
          @Override
          public void onNewResultImpl(DataSource<T> dataSource) {
            //......
          }
          @Override
          public void onFailureImpl(DataSource<T> dataSource) {
            //...
          }
          @Override
          public void onProgressUpdate(DataSource<T> dataSource) {
            //......
          }
        };
    mDataSource.subscribe(dataSubscriber, mUiThreadImmediateExecutor);
```

其实上一节我们就已经知道，在new datasource 实例时就会通过 producer开始图片的加载流程。这里的subscribe是为了监听流程处理的进度以及结果。下面为datasource的构造函数:

```
protected AbstractProducerToDataSourceAdapter() {
    producer.produceResults(createConsumer(), settableProducerContext);
}
```

那么接下来追踪一下Fresco是如何使用`Producer`控制图片加载的流程的。根据上一节的阅读，先来找到具体的Producer的实例：

>ProducerSequenceFactory.java

```
public Producer<CloseableReference<CloseableImage>> getDecodedImageProducerSequence(
      ImageRequest imageRequest) {
    Producer<CloseableReference<CloseableImage>> pipelineSequence =
        getBasicDecodedImageSequence(imageRequest);
    .....
}

private Producer<CloseableReference<CloseableImage>> getBasicDecodedImageSequence(
      ImageRequest imageRequest) {
    Preconditions.checkNotNull(imageRequest);

    Uri uri = imageRequest.getSourceUri();
    Preconditions.checkNotNull(uri, "Uri is null.");

    switch (imageRequest.getSourceUriType()) {
      case SOURCE_TYPE_NETWORK:
        return getNetworkFetchSequence();
      case SOURCE_TYPE_LOCAL_VIDEO_FILE:
        return getLocalVideoFileFetchSequence();
      case SOURCE_TYPE_LOCAL_IMAGE_FILE:
     ...... 
  }

```

即对于不同的图片请求使用不同的producer去加载。这里看一下对于网络请求加载的Producer:`getNetworkFetchSequence()`

```
  /**
    * swallow result if prefetch -> bitmap cache get ->
    * background thread hand-off -> multiplex -> bitmap cache -> decode -> multiplex ->
    * encoded cache -> disk cache -> (webp transcode) -> network fetch.
    */
  private synchronized Producer<CloseableReference<CloseableImage>> getNetworkFetchSequence() {
    if (mNetworkFetchSequence == null) {
      mNetworkFetchSequence =
          newBitmapCacheGetToDecodeSequence(getCommonNetworkFetchToEncodedMemorySequence());
    }
    return mNetworkFetchSequence;
  }

  /**
   * multiplex -> encoded cache -> disk cache -> (webp transcode) -> network fetch.
   */
  private synchronized Producer<EncodedImage> getCommonNetworkFetchToEncodedMemorySequence() {
    if (mCommonNetworkFetchToEncodedMemorySequence == null) {
      Producer<EncodedImage> inputProducer =
          newEncodedCacheMultiplexToTranscodeSequence(
              mProducerFactory.newNetworkFetchProducer(mNetworkFetcher));
      mCommonNetworkFetchToEncodedMemorySequence =
          ProducerFactory.newAddImageTransformMetaDataProducer(inputProducer);

      mCommonNetworkFetchToEncodedMemorySequence =
          mProducerFactory.newResizeAndRotateProducer(
              mCommonNetworkFetchToEncodedMemorySequence,
              mResizeAndRotateEnabledForNetwork,
              mUseDownsamplingRatio);
    }
    return mCommonNetworkFetchToEncodedMemorySequence;
  }
```

上面的注释大致解释了Fresco在获取网络图片时的一个处理流程，也可以看出每一个流程由一个Sequence代表。每一个Sequence由多个producer拼接而成，并组成一个图片加载流程序列。


## Producer与Sequence的理解

1. 在Fresco中，图片处理的每一步可以简称为一个Producer。
2. 每一个Produer都可以指定一个InputProducer，**Produer将会调用这个inputProduer的`produceResults()`方法，并wrapper一个consumer作为回调监听。**
3. Fresco中并没有Sequence这个对象。Sequence的概念是虚拟的，Produer通过拼接可以组成Sequence。

大致可以使用下图表示：

![](http://7xrbxa.com1.z0.glb.clouddn.com/galaxywarshipProducerAndSequence.png)






