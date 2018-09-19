---
title: Fresco-EncodeMemory和DiskCache
date: 2018/6/28
tags: 
  - Fresco
  - susion
---

>根据上节分析还剩下这几步

```
-> Encode Memory -> （disk cache） -> (webp tracscode) ->  Net Get
```

在看`EncodedMemoryCacheProducer`之前，先来回忆一下Fresco中的缓存层级:
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

即`EncodedMemoryCacheProducer`属于第三等级，在Fresco在直接加载图片时，如果从内存缓存中获取不到，就会去这里获得。

## EncodedMemoryCacheProducer

### `produceResults`

我们已经知道`produceResults`方法是图片处理流程到达这个Producer时的入口方法。 `EncodedMemoryCacheProducer`在`produceResults`方法中主要做了3件事：

1. 判断缓存中是否已经存在，如果存在直接返回

```
    CloseableReference<PooledByteBuffer> cachedReference = mMemoryCache.get(cacheKey);
    if (cachedReference != null) {
        EncodedImage cachedEncodedImage = new EncodedImage(cachedReference);
        ......
        consumer.onNewResult(cachedEncodedImage, Consumer.IS_LAST);
        return;
    }
```

2. 判断当前请求等级是否大于 `RequestLevel.ENCODED_MEMORY_CACHE`。如果大于或等于则直接返回(用户只希望在获取图片时从 `BITMAP_MEMORY_CACHE`或者`ENCODED_MEMORY_CACHE`中获取)

```
    if (producerContext.getLowestPermittedRequestLevel().getValue() >=
        ImageRequest.RequestLevel.ENCODED_MEMORY_CACHE.getValue()) {
        consumer.onNewResult(null, Consumer.IS_LAST); //上一步已经从缓存中取了，这时肯定是没结果的。
        return;
    }
```

3. 监听图片处理流程结果，获取结果后缓存 EncodedImage

```
    Consumer consumerOfInputProducer =new EncodedMemoryCacheConsumer(consumer, mMemoryCache, cacheKey);
    mInputProducer.produceResults(consumerOfInputProducer, producerContext);

    private static class EncodedMemoryCacheConsumerextends DelegatingConsumer<EncodedImage, EncodedImage> {

    ......

    @Override
    public void onNewResultImpl(EncodedImage newResult, @Status int status) {
      // cache and forward the last result
      CloseableReference<PooledByteBuffer> ref = newResult.getByteBufferRef();
      if (ref != null) {
        CloseableReference<PooledByteBuffer> cachedResult;
        cachedResult = mMemoryCache.cache(mRequestedCacheKey, ref);

        if (cachedResult != null) {
          EncodedImage cachedEncodedImage;
          cachedEncodedImage = new EncodedImage(cachedResult);
          cachedEncodedImage.copyMetaDataFrom(newResult);
          getConsumer().onNewResult(cachedEncodedImage, status);
        }
      }
      getConsumer().onNewResult(newResult, status);
    }
  }
```


## 磁盘缓存

>Fresco中的磁盘缓存分为两种形式:

```
  private Producer<EncodedImage> newDiskCacheSequence(Producer<EncodedImage> inputProducer) {
    Producer<EncodedImage> cacheWriteProducer;
    if (mPartialImageCachingEnabled) {
      Producer<EncodedImage> partialDiskCacheProducer =
          mProducerFactory.newPartialDiskCacheProducer(inputProducer);
      cacheWriteProducer = mProducerFactory.newDiskCacheWriteProducer(partialDiskCacheProducer);
    } else {
      cacheWriteProducer = mProducerFactory.newDiskCacheWriteProducer(inputProducer);
    }
    Producer<EncodedImage> mediaVariationsProducer =
        mProducerFactory.newMediaVariationsProducer(cacheWriteProducer);
    return mProducerFactory.newDiskCacheReadProducer(mediaVariationsProducer);
  }
```

这里先看一下在不开启部分缓存的情况下(`mPartialImageCachingEnabled`为false):


### DiskCacheReadProducer

> 逻辑还是很简单的，磁盘获取，得到结果，返回:

```
  public void produceResults(final Consumer<EncodedImage> consumer,final ProducerContext producerContext) {

    //根据请求获取缓存key
    final CacheKey cacheKey = mCacheKeyFactory.getEncodedCacheKey(imageRequest, producerContext.getCallerContext());
    //获取磁盘缓存
    final BufferedDiskCache preferredCache = isSmallRequest ?mSmallImageBufferedDiskCache : mDefaultBufferedDiskCache;
    //获取缓存的任务
    final Task<EncodedImage> diskLookupTask = preferredCache.get(cacheKey, isCancelled);
    //获取缓存后的操作 : 返回结果
    final Continuation<EncodedImage, Void> continuation =onFinishDiskReads(consumer, producerContext);
    //缓存获取
    diskLookupTask.continueWith(continuation);
  }
```

### DiskCacheWriteProducer

> 这个Producer作为 `DiskCacheReadProducer`的 inputProduer。负责磁盘缓存的写入。直接看WrapperConsumer:

```
private static class DiskCacheWriteConsumer extends DelegatingConsumer<EncodedImage, EncodedImage> {

    private final ProducerContext mProducerContext;
    private final BufferedDiskCache mDefaultBufferedDiskCache;
    private final BufferedDiskCache mSmallImageBufferedDiskCache;
    private final CacheKeyFactory mCacheKeyFactory;

    private DiskCacheWriteConsumer(
        final Consumer<EncodedImage> consumer,
        final ProducerContext producerContext,
        final BufferedDiskCache defaultBufferedDiskCache,
        final BufferedDiskCache smallImageBufferedDiskCache,
        final CacheKeyFactory cacheKeyFactory) {
      super(consumer);
      mProducerContext = producerContext;
      mDefaultBufferedDiskCache = defaultBufferedDiskCache;
      mSmallImageBufferedDiskCache = smallImageBufferedDiskCache;
      mCacheKeyFactory = cacheKeyFactory;
    }

    @Override
    public void onNewResultImpl(EncodedImage newResult, @Status int status) {
      ...... 
      final ImageRequest imageRequest = mProducerContext.getImageRequest();
      final CacheKey cacheKey = mCacheKeyFactory.getEncodedCacheKey(imageRequest, mProducerContext.getCallerContext());

      if (imageRequest.getCacheChoice() == ImageRequest.CacheChoice.SMALL) {
        mSmallImageBufferedDiskCache.put(cacheKey, newResult);
      } else {
        mDefaultBufferedDiskCache.put(cacheKey, newResult);
      }
      getConsumer().onNewResult(newResult, status);
}
```



