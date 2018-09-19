---
title: Fresco图片加载流程架构设计的艺术
date: 2018/6/28
tags: 
  - Fresco
  - susion
---

>Fresco作为Android中一款优秀的图片加载框架，开发者在使用时仅仅通过一行代码就可以完成网络图片的加载与设置。但框架本身在背后却是做了很多操作的: 支持更多特性、图片加载速度的优化等。本文来看一下Fresco是如何设计整个图片加载流程的。

在分析Fresco中图片加载流程之前，我们先来看一下在Fresco中加载一张网络图片，Fresco背后都是经过哪些流程：

![](http://7xrbxa.com1.z0.glb.clouddn.com/galaxywarshipnet_image_load_process.png)

从上图可以看出，Fresco对于一次图片加载是经过了非常多的步骤的。上图标识了两个流向：

### 蓝色线的图片加载流程

在这个流程中，Fresco从网路获取一个`EncodeImage`，然后把这个图片缓存到磁盘、内存、旋转后，然后经过Decode后获得`CloseableImage`。再把这个图片缓存到内存，最后交给我们。

### 灰色线的图片加载流程

和蓝色线的流程相反，先去内存中获取，最后的最后再到网络加载图片。

# Fresco的图片加载流程

> Fresco在加载图片时其实走的是 `灰色`。 但对于第一次加载网络图片，实际流程却像是`蓝色`的流向。 当我们发起一次图片请求时：

```
1. 去内存缓存中去读，如果有的话直接返回。
2. 切换到后台线程。
3. 合并同一个请求，一旦一个请求获得返回结果，这些请求都获得返回结果
4. ”内存写“ 占一个流程坑位 （监听后续图片请求结果，得到结果缓存到内存）
5. ”Decode“ 占一个流程坑位 (监听到图片请求结果，做图片的解码)
6. “Resize Rotate” 占一个流程坑位 (监听到图片请求结果，做图片的Resize和Rotate)
7. ”AddMetaData“ 占一个流程坑位 (监听到图片请求结果，做add meta data)
8. ”EncodeMemory“ 占一个流程坑位 (监听到图片请求结果, 把EncodeImage缓存到内存)， 如果有结果直接返回
9. ”Disk Cache“ 占一个流程坑位（监听图片请求结果， 把EncodeIamge缓存到磁盘）， 如果有结果直接返回
10. ”Web TransCode“  占一个流程坑位（监听图片请求结果，做WebP的转码）
11. 去网络获取
```

那么大致了解完Fresco图片的处理流程后，可能会有一些疑问:

- Fresco是如何组织上面这些步骤为一个图片加载流程的 ？
- 一个流程如何获得上一个流程的处理结果？
- 如何在任意一个流程结束图片加载，返回结果？
- ...

>ok， 接下来就看一些这些在Fresco中是怎么实现的。

### Producer
在Fresco中一个`Producer`代表图片处理的一步，具有很好的可复用性，是组成图片处理的基本单位。比如内存读`BitmapMemoryCacheGetProducer`、 比如解码`DecodeProducer`。 看一下`Producer`的定义:

```
/**
 * Building block for image processing in the image pipeline.
 * <p> Execution of image request consists of multiple different tasks such as network fetch,
 * disk caching, memory caching, decoding, applying transformations etc. Producer<T> represents
 * single task whose result is an instance of T. Breaking entire request into sequence of
 * Producers allows us to construct different requests while reusing the same blocks.
public interface Producer<T> {
  void produceResults(Consumer<T> consumer, ProducerContext context);
}
```

`produceResults`可以简单理解为此`Producer`的处理流程开始。在Producer的具体实现中，每一个`Producer`都有一个`inputProduer`。如果这个`Producer`完成自己的逻辑后，会调用`inputProducer`的`produceResults`。

那`Consumer`与`ProducerContext`又代表什么呢？

### Consumer
简单理解就是个监听器。监听`Producer`产生的结果。看一下接口定义就明白了:

```
public interface Consumer<T> {
  ....
  void onNewResult(T newResult, @Status int status);

  void onFailure(Throwable t);

  void onCancellation();

  void onProgressUpdate(float progress);
}
```
### ProducerContext
如其名，一个图片加载流程中的上下文，可以获取请求的参数等等。


## Fresco中的两种Producer

*这里个人把Producer分成两种: `前置Producer` 与 `后置Produer`。  （名字是自己起的，官方并没有这个概念*）

### 前置Produer
简单的说，就是`produceResults`方法做自己的处理，比如`BitmapMemoryCacheGetProducer`,我们来看一下它的`produceResults`方法

```
  @Override
  public void produceResults(final Consumer<CloseableReference<CloseableImage>> consumer,final ProducerContext producerContext) {
    ...
    final CacheKey cacheKey = mCacheKeyFactory.getBitmapCacheKey(imageRequest, callerContext);
    CloseableReference<CloseableImage> cachedReference = mMemoryCache.get(cacheKey);

    if (cachedReference != null) {
     .....
      consumer.onNewResult(cachedReference, BaseConsumer.simpleStatusForIsLast(isFinal));
      cachedReference.close();
        return;
    }
    ...
    mInputProducer.produceResults(wrappedConsumer, producerContext);
  }
```

上面只是截取了最主要的逻辑:从缓存中获取图片，如果图片已经缓存则`consumer.onNewResult()`，即返回结果，然后返回。否则让`inputProduer`继续处理。

### 后置Producer
直接对比看`DecodeProducer`

```
  @Override
  public void produceResults(
      final Consumer<CloseableReference<CloseableImage>> consumer,
      final ProducerContext producerContext) {
    final ImageRequest imageRequest = producerContext.getImageRequest();
    ProgressiveDecoder progressiveDecoder = new NetworkImagesProgressiveDecoder(...);
    mInputProducer.produceResults(progressiveDecoder, producerContext);
  }

  private abstract class ProgressiveDecoder extends DelegatingConsumer<EncodedImage, CloseableReference<CloseableImage>> {
    @Override
    public void onNewResultImpl(EncodedImage newResult, @Status int status) {
        .....
        doDecode(newResult, status) 
    }
  }
```

即后置`Produer`会监听下一步的处理结果，来做处理。

## 串连 Procuder

> 上面介绍了Fresco中图片处理流程的基本单元`Producer`。那么这些Producer是如何组成Fresco图片处理流程的呢？ 

### ProducerSequence

在Fresco中一个`ProducerSequence`由许多`Producer`组成，代表着一种图片加载流程，比如`NetworkFetchSequence`,就是Fresco网络图片加载的流程。

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

其实从这个方法是看不出Fresco如何串联`Producer`的，具体看一下`newBitmapCacheGetToBitmapCacheSequence()`

```
  /**
   * Bitmap cache get -> thread hand off -> multiplex -> bitmap cache
   * @param inputProducer producer providing the input to the bitmap cache
   * @return bitmap cache get to bitmap cache sequence
   */
  private Producer<CloseableReference<CloseableImage>> newBitmapCacheGetToBitmapCacheSequence(Producer<CloseableReference<CloseableImage>> inputProducer) {
    BitmapMemoryCacheProducer bitmapMemoryCacheProducer = mProducerFactory.newBitmapMemoryCacheProducer(inputProducer);

    BitmapMemoryCacheKeyMultiplexProducer bitmapKeyMultiplexProducer = mProducerFactory.newBitmapMemoryCacheKeyMultiplexProducer(bitmapMemoryCacheProducer);

    ThreadHandoffProducer<CloseableReference<CloseableImage>> threadHandoffProducer = mProducerFactory.newBackgroundThreadHandoffProducer(bitmapKeyMultiplexProducer, mThreadHandoffProducerQueue);

    return mProducerFactory.newBitmapMemoryCacheGetProducer(threadHandoffProducer);
  }
```

上面的代码直接解释就是：

> `BitmapMemoryCacheProducer` -> 传给(作为inputProducer)-> `BitmapMemoryCacheKeyMultiplexProducer` -> 传给(作为inputProducer)-> `ThreadHandoffProducer` -> 传给(作为inputProducer)-> `BitmapMemoryCacheGetProducer`-> 返回

#### BitmapMemoryCacheGetProducer

> 其实前面我们已经看过这个`Producer`了，称为“前置Producer”。因此不看代码了，我们看一下它的处理流程可能更好明白：

![](http://7xrbxa.com1.z0.glb.clouddn.com/galaxywarshipBitmapMemoryCacheGetProducerProcess.png)

### DecodeProducer

> 同样看一张图

![](http://7xrbxa.com1.z0.glb.clouddn.com/galaxywarshipDecodeProducerProcess.png)


>相信通过上面两张图，已经大致解释清楚了Fresco中的*前置Procuder*与*后置Producer*

## ProducerSequence模型图

>经过上面的认识，Producer中的图片加载流程模型可以使用下图表示:

![](http://7xrbxa.com1.z0.glb.clouddn.com/galaxywarshipProducerSequenceModel.png)


# END

> 上面我们看完了Fresco是如何组织图片加载流程的。

1. Fresco利用`Producer`的特性，巧妙的将图片加载流程串联起来。
2. 一个`Producer`可以接受另一个`Produer`的反馈，可以逻辑很清晰的做到从网路加载了图片，然后缓存到内存这种设计。
3. 每个`Producer`专注于自己的职责。
3. Fresco的`ProducerSequence`分的很细，可以很好的复用。


















