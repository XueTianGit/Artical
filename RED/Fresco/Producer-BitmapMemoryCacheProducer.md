---
title: Producer-BitmapMemoryCacheProducer
date: 2018/5/06
tags: 
  - Fresco
  - susion
---

> 前面几节大致了解了Fresco中的`Producer`，并分析了`NetworkFetchProducer`。接下来就大致分析一下整个图片加载的流程。还是从构建网络获取图片Sequence的地方开始：

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
  
```

上面的注释，其实已经说明了网络获取图片的流程，总的来说呢，就是先去缓存获取，然后磁盘，再最后网络，至于像`encoded cache`这种步骤，我还不是很明白具体是怎么操作，为了什么，所以接下来要通过源码再慢慢了解了。

先看第一个入口的Producer: `BitmapMemoryCacheGetProducer` :

```
  public BitmapMemoryCacheGetProducer newBitmapMemoryCacheGetProducer(
      Producer<CloseableReference<CloseableImage>> inputProducer) {
    return new BitmapMemoryCacheGetProducer(mBitmapMemoryCache, mCacheKeyFactory, inputProducer);
  }
```

`BitmapMemoryCacheGetProducer`这个类实现很简单，官方注释为： Bitmap memory cache producer that is read-only.
即，仅仅去Bitmap Cache读取是否已经有缓存的操作，而不做其他操作。其实我们通过前面对Producer的认识可知，Produer可以通过wrapper consumer 来监听它的 inputProducer 的处理结果，而`BitmapMemoryCacheGetProducer`没有wrapper consumer

```
  @Override
  protected Consumer<CloseableReference<CloseableImage>> wrapConsumer(
      final Consumer<CloseableReference<CloseableImage>> consumer,
      final CacheKey cacheKey) {
    // since this cache is read-only, we can pass our consumer directly to the next producer
    return consumer;
  }
```

这个类的父类是`BitmapMemoryCacheProducer`,其实其父类是wrapper consumer， 并对 input producer得到的结果进行缓存的。

即 `BitmapMemoryCacheGetProducer`的处理逻辑时，直接去缓存中读取图片，如果没有则将下面的事情交给它的input producer处理，并且不对返回结果做任何处理。看一下其 inputProducer 是什么：

```
/**
   * Bitmap cache get -> thread hand off -> multiplex -> bitmap cache
   * @param inputProducer producer providing the input to the bitmap cache
   * @return bitmap cache get to bitmap cache sequence
   */
  private Producer<CloseableReference<CloseableImage>> newBitmapCacheGetToBitmapCacheSequence(
      Producer<CloseableReference<CloseableImage>> inputProducer) {

    BitmapMemoryCacheProducer bitmapMemoryCacheProducer =
        mProducerFactory.newBitmapMemoryCacheProducer(inputProducer);

    BitmapMemoryCacheKeyMultiplexProducer bitmapKeyMultiplexProducer =
        mProducerFactory.newBitmapMemoryCacheKeyMultiplexProducer(bitmapMemoryCacheProducer);

    ThreadHandoffProducer<CloseableReference<CloseableImage>> threadHandoffProducer =
        mProducerFactory.newBackgroundThreadHandoffProducer(
            bitmapKeyMultiplexProducer,
            mThreadHandoffProducerQueue);

    return mProducerFactory.newBitmapMemoryCacheGetProducer(threadHandoffProducer);
  }

```

即 `BitmapMemoryCacheGetProducer`的input Produer是 `ThreadHandoffProducer` 然后 -> `BitmapMemoryCacheKeyMultiplexProducer` 然后 -> `BitmapMemoryCacheProducer`

`ThreadHandoffProducer`和`BitmapMemoryCacheKeyMultiplexProducer`下一节再看，这里先看一下 `BitmapMemoryCacheProducer` ： 

## BitmapMemoryCacheProducer

- 从内存缓存获取

这个`Producer`前面已经看过一些：根据图片请求和当前的请求上下文从内存缓存中查看是否已经有缓存的图片，如果有则直接返回结果：

```
  final CacheKey cacheKey = mCacheKeyFactory.getBitmapCacheKey(imageRequest, callerContext);
    CloseableReference<CloseableImage> cachedReference = mMemoryCache.get(cacheKey);

    if (cachedReference != null) {
      boolean isFinal = cachedReference.get().getQualityInfo().isOfFullQuality();
      if (isFinal) {
        listener.onProducerFinishWithSuccess(
            requestId,
            getProducerName(),
            listener.requiresExtraMap(requestId)
                ? ImmutableMap.of(EXTRA_CACHED_VALUE_FOUND, "true")
                : null);
        listener.onUltimateProducerReached(requestId, getProducerName(), true);
        consumer.onProgressUpdate(1f);
      }
      consumer.onNewResult(cachedReference, BaseConsumer.simpleStatusForIsLast(isFinal));
      cachedReference.close();
      if (isFinal) {
        return;
      }
    }
```

`consumer.onNewResult`根据我们前面了解是用来通知`DataSourceSubscriber`的，那这里的`listener`又是什么东西呢？
其实经过一路追踪可以发现`ImagePipeline`在构造图片请求时，会传递给`SettableProducerContext`这个监听，

```
 SettableProducerContext settableProducerContext = new SettableProducerContext(
          imageRequest,
          generateUniqueFutureId(),
          requestListener,
          callerContext,
          lowestPermittedRequestLevel,
        /* isPrefetch */ true,
        /* isIntermediateResultExpected */ false,
          priority);

  private RequestListener getRequestListenerForRequest(ImageRequest imageRequest) {
    if (imageRequest.getRequestListener() == null) {
      return mRequestListener;
    }
    return new ForwardingRequestListener(mRequestListener, imageRequest.getRequestListener());
  }
```
所以这个listener就是 `ImagePipeline` 用来监听图片处理进度的监听，同时，我们我们也可以给`imageRequest`设置监听，监听这些事件。`ForwardingRequestListener`这个对象就是一个维护监听的容器，并调用其所维护的listeners的方法。


- 图片RequestLevel判断

Fresco对于图片请求有灵活的设置，如果我们对于一个请求，给它设置RequestLevel为仅仅从内存获取，那么就不会去磁盘或者网络获取，Fresco的RequestLevel如下：
```
/**
   * Level down to we are willing to go in order to find an image. E.g., we might only want to go
   * down to bitmap memory cache, and not check the disk cache or do a full fetch.
   */
  public enum RequestLevel {
    /* Fetch (from the network or local storage) */
    FULL_FETCH(1),

    /* Disk caching */
    DISK_CACHE(2),

    /* Encoded memory caching */
    ENCODED_MEMORY_CACHE(3),

    /* Bitmap caching */
    BITMAP_MEMORY_CACHE(4);
  }
```

对于`BitmapMemoryCacheGetProducer`来说，如果设置的RequestLevel大于4，那么如果缓存中没有的话，那么这次图片请求就结束了：

```
  if (producerContext.getLowestPermittedRequestLevel().getValue() >=
        ImageRequest.RequestLevel.BITMAP_MEMORY_CACHE.getValue()) {
      listener.onProducerFinishWithSuccess(
          requestId,
          getProducerName(),
          listener.requiresExtraMap(requestId)
              ? ImmutableMap.of(EXTRA_CACHED_VALUE_FOUND, "false")
              : null);
      listener.onUltimateProducerReached(requestId, getProducerName(), false);
      consumer.onNewResult(null, Consumer.IS_LAST);
      return;
    }
```

如果RequestLevel小于4，那么`BitmapMemoryCacheGetProducer`会把接下来继续获取图片的任务交给它的`inputProducer`，并监听`inputProducer`的处理结果：

```
 Consumer<CloseableReference<CloseableImage>> wrappedConsumer = wrapConsumer(consumer, cacheKey);
 mInputProducer.produceResults(wrappedConsumer, producerContext);
```

- 图片请求结果的回调处理

首先在Fresco中，对于图片的处理是否决定做一些操作是有一些状态表示的：

`Consumer.java`

```
/** Status flag used by producers and consumers to supply additional information. */
  @Retention(SOURCE)
  @IntDef(
    flag = true,
    value = {
      IS_LAST,  //图片请求的最后一步后获取的结果，不会有更多处理的
      DO_NOT_CACHE_ENCODED, //图片请求不应该被缓存，即使是 IS_LAST
      IS_PLACEHOLDER, //图片请求是否是 PLACEHOLDER
      IS_PARTIAL_RESULT, //图片请求是否是部分图片，其实这代表着这次图片请求失败或者是被取消，反正结果是不能用的
      IS_RESIZING_DONE, //图片是否是被resize过
    }
  )
```

下面来看一下对于监听到图片处理结果是如何做的：

```
  public void onNewResultImpl(
          CloseableReference<CloseableImage> newResult,
          @Status int status) {
        final boolean isLast = isLast(status);
        // ignore invalid intermediate results and forward the null result if last
        if (newResult == null) {
          if (isLast) {
            getConsumer().onNewResult(null, status);
          }
          return;
        }
        // stateful and partial results cannot be cached and are just forwarded
        if (newResult.get().isStateful() || statusHasFlag(status, IS_PARTIAL_RESULT)) {
          getConsumer().onNewResult(newResult, status);
          return;
        }
        // if the intermediate result is not of a better quality than the cached result,
        // forward the already cached result and don't cache the new result.
        if (!isLast) {
          CloseableReference<CloseableImage> currentCachedResult = mMemoryCache.get(cacheKey);
          if (currentCachedResult != null) {
            try {
              QualityInfo newInfo = newResult.get().getQualityInfo();
              QualityInfo cachedInfo = currentCachedResult.get().getQualityInfo();
              if (cachedInfo.isOfFullQuality() || cachedInfo.getQuality() >= newInfo.getQuality()) {
                getConsumer().onNewResult(currentCachedResult, status);
                return;
              }
            } finally {
              CloseableReference.closeSafely(currentCachedResult);
            }
          }
        }
        // cache and forward the new result
        CloseableReference<CloseableImage> newCachedResult =
            mMemoryCache.cache(cacheKey, newResult);
        try {
          if (isLast) {
            getConsumer().onProgressUpdate(1f);
          }
          getConsumer().onNewResult(
              (newCachedResult != null) ? newCachedResult : newResult, status);
        } finally {
          CloseableReference.closeSafely(newCachedResult);
        }
      }
    };
```

1. 如果图片result是 null， 并且是`IS_LAST`,则回调`Consumer`监听
2. 如果图片result是 partial results ，则这次图片请求并不能被缓存，回调`Consumer`监听
3. 如果这次图片result没有已经缓存的result好，则直接把原来缓存的更好的图片回调`Consumer`监听
4. 缓存结果，并且回调`Consumer`监听


>总的处理流程如下图：

![](http://7xrbxa.com1.z0.glb.clouddn.com/galaxywarshipBitmapCacheGetProducer.png)