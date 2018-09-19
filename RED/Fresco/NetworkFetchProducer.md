---
title: 图片加载之NetWorkFetchProducer
date: 2018/4/23
tags: 
  - Fresco
  - susion
---

> 上一节分析了Producer与Sequence，大致了解了Fresco图片处理的流程。接下来从网络图片获取这一步继续追踪源码。
> 首先Fresco在获取图片时通过前面的分析是不会直接去网络获取图片的，但是对于第一次图片加载肯定是从网络获取的，因此这里从网络获取图片开始。

>先看一下注释：

```
/**
 * A producer to actually fetch images from the network.
 *
 * <p> Downloaded bytes may be passed to the consumer as they are downloaded, but not more often
 * than {@link #TIME_BETWEEN_PARTIAL_RESULTS_MS}.

 * <p>Clients should provide an instance of {@link NetworkFetcher} to make use of their networking
 * stack. Use {@link HttpUrlConnectionNetworkFetcher} as a model.
 */
public class NetworkFetchProducer implements Producer<EncodedImage>
```

大致明白这个类主要承担网络图片下载的控制工作，并将下载中的事件反馈给外部。看一下`produceResults`方法。

```
@Override
  public void produceResults(Consumer<EncodedImage> consumer, ProducerContext context) {
    context.getListener().onProducerStart(context.getId(), PRODUCER_NAME);
    final FetchState fetchState = mNetworkFetcher.createFetchState(consumer, context);
    mNetworkFetcher.fetch(
        fetchState, new NetworkFetcher.Callback() {
          @Override
          public void onResponse(InputStream response, int responseLength) throws IOException {
            NetworkFetchProducer.this.onResponse(fetchState, response, responseLength);
          }

          @Override
          public void onFailure(Throwable throwable) {
            NetworkFetchProducer.this.onFailure(fetchState, throwable);
          }

          @Override
          public void onCancellation() {
            NetworkFetchProducer.this.onCancellation(fetchState);
          }
        });
  }
```

即创建了`FetchState`，然后回调`mNetworkFetcher`的结果。`FetchState`是用来 encapsulate(包装) the state of one network fetch。`NetworkFetcher`则是网络下载的实现类，具体指`HttpUrlConnectionNetworkFetcher`.

> 先来看一下，对于拿到网络下载的结果如何传递给外部的？ 在 `handleFinalResult()`方法中有下面方法：

```
  private void notifyConsumer(
      PooledByteBufferOutputStream pooledOutputStream,
      @Consumer.Status int status,
      @Nullable BytesRange responseBytesRange,
      Consumer<EncodedImage> consumer) {
      //...
      consumer.onNewResult(encodedImage, status);
      //...
  }
```

这里的consumer其实就是`produceResults`的consumer参数。前面已经知道`Consumer`其实就是一个Sequence中传递Sequence处理结果的回调。对`Consumer`追本溯源，其实来自我们前面了解的:`AbstractProducerToDataSourceAdapter`
我们前面看到这个类在被构造时会开始使用`producer`进入图片处理流程，其实对于producer的`consumer`就是这个类创建的：

```
  private Consumer<T> createConsumer() {
    return new BaseConsumer<T>() {
      @Override
      protected void onNewResultImpl(@Nullable T newResult, @Status int status) {
        AbstractProducerToDataSourceAdapter.this.onNewResultImpl(newResult, status);
      }

      @Override
      protected void onFailureImpl(Throwable throwable) {
        AbstractProducerToDataSourceAdapter.this.onFailureImpl(throwable);
      }

      @Override
      protected void onCancellationImpl() {
        AbstractProducerToDataSourceAdapter.this.onCancellationImpl();
      }

      @Override
      protected void onProgressUpdateImpl(float progress) {
        AbstractProducerToDataSourceAdapter.this.setProgress(progress);
      }
    };
  }
```

到这里对`AbstractProducerToDataSourceAdapter`的作用可能还是有些疑惑，那么看一下`setProgress(progress)`

```
protected boolean setProgress(float progress) {
    boolean result = setProgressInternal(progress);
    if (result) {
      notifyProgressUpdate();
    }
    return result;
}

protected void notifyProgressUpdate() {
    for (Pair<DataSubscriber<T>, Executor> pair : mSubscribers) {
      final DataSubscriber<T> subscriber = pair.first;
      Executor executor = pair.second;
      executor.execute(
          new Runnable() {
            @Override
            public void run() {
              subscriber.onProgressUpdate(AbstractDataSource.this);
            }
          });
    }
}
```

到这就可以明白一些头绪了：对于一个`DataSource`可能会有多个结果`DataSubscriber`。`AbstractProducerToDataSourceAdapter`的一个作用就是把图片处理结果回调给这些`DataSubscriber`,并且可以指定回调结果的线程。那再看一遍图片开始加载时的`DataSubscriber`

>AbstractDraweeController.java

```
 final DataSubscriber<T> dataSubscriber =
        new BaseDataSubscriber<T>() {
         //...
          @Override
          public void onProgressUpdate(DataSource<T> dataSource) {
            boolean isFinished = dataSource.isFinished();
            float progress = dataSource.getProgress();
            onProgressUpdateInternal(id, dataSource, progress, isFinished);
          }
        };
    mDataSource.subscribe(dataSubscriber, mUiThreadImmediateExecutor);
```

**上面大致明白了`NetworkFetchProducer`对于图片处理的结果如何传递给外部，下面具体看一下细节， 即其两个基本构成组件：ByteArrayPool 与 NetworkFetcher**

## ByteArrayPool

`NetworkFetchProducer.this.onResponse`:

```
 protected void onResponse(
      FetchState fetchState, InputStream responseData, int responseContentLength)
      throws IOException {

    final PooledByteBufferOutputStream pooledOutputStream;
    if (responseContentLength > 0) {
      pooledOutputStream = mPooledByteBufferFactory.newOutputStream(responseContentLength);
    } else {
      pooledOutputStream = mPooledByteBufferFactory.newOutputStream();
    }
    final byte[] ioArray = mByteArrayPool.get(READ_SIZE);
    try {
      int length;
      while ((length = responseData.read(ioArray)) >= 0) {
        if (length > 0) {
          pooledOutputStream.write(ioArray, 0, length);
          maybeHandleIntermediateResult(pooledOutputStream, fetchState);
          float progress = calculateProgress(pooledOutputStream.size(), responseContentLength);
          fetchState.getConsumer().onProgressUpdate(progress);
        }
      }
      mNetworkFetcher.onFetchCompletion(fetchState, pooledOutputStream.size());
      handleFinalResult(pooledOutputStream, fetchState);
    } finally {
      mByteArrayPool.release(ioArray);
      pooledOutputStream.close();
    }
  }
```

从上面可以看到：
1. 利用`PooledByteBufferFactory`根据下载内容的大小new了一个`OutputStream`。
2. 从`ByteArrayPool`中获得一个下载字节缓冲区`ioArray`。
3. 把内容读到`ioArray`，写到输出流中。

恩，很正常的文件读写步骤。但这里看一下这个听名字不简单的`ByteArrayPool`：

追踪源码，这个类的具体实现类是`GenericByteArrayPool`,看一下官方注释：

```
/**
 * A pool of byte arrays.
 * The pool manages a number of byte arrays of a predefined set of sizes. This set of sizes is
 * typically, but not required to be, based on powers of 2.
 * The pool supports a get/release paradigm.
 * On a get request, the pool attempts to find an existing byte array whose size
 * is at least as big as the requested size.
 * On a release request, the pool adds the byte array to the appropriate bucket.
 * This byte array can then be used for a subsequent get request.
 */
@ThreadSafe
public class GenericByteArrayPool extends BasePool<byte[]> implements ByteArrayPool
```

总的来说是一个管理许多字节数组的地方，这些字节数组可以被重复利用。查看其父类`BasePool`,可以知道这些字节数组都被保存在map中。map的value其实是一个使用队列建模的链表, 看一下`BasePool`源码：

```
A base pool class that manages a pool of values (of type V). <p>
The pool is organized as a map. Each entry in the map is a free-list (modeled by a queue) of
entries for a given size.
```

其实使用的就是这个数据结构：`final SparseArray<Bucket<V>> mBuckets;`， `Bucket`就是一个使用队列建模的列表，用来保存数据。对用`GenericByteArrayPool`，我们可以知道的就是我们可以从中获取我们**想得到的大小的字节数组**

**至于为什么要有这个`BasePool`,大概是为了避免频繁分配内存和更好的管理内存**, `BasePool`还有一个特点是它可以通过`MemoryTrimmableRegistry`观察系统低内存事件，当系统发出低内存事件时，它会释放一些空间。

`BasePool`还有很多子类，这里暂时不去看了，只是先明白我们可以从`GenericByteArrayPool`获取指定差不多大小的字节数组用来保存下载的数据就可以了。


## NetworkFetcher

>前面已经知道，这个类是 NetworkFetchProducer 从网络获取图片的基本组件，它也是一个接口，下面具体看一下实现类 `HttpUrlConnectionNetworkFetcher`

这个类其实没有太多去深究的点，总结一下：

- 在`FetchState`中获取这次网络任务的上下文
- 使用 `HttpURLConnection` 实现网络下载。
- 使用`ExecutorService` 实现线程切换
- 存在一个重定向下载机制


到这里我们已经分析完成`NetworkFetchProducer`的大致实现逻辑：通过 `NetworkFetcher`和`ByteArrayPool`这两个基本组件，获取网络图片，封装成`EncodedImage`,然后传递给 `Consumer`。总结一下大致如下图：


![](http://7xrbxa.com1.z0.glb.clouddn.com/galaxywarshipnetwordfetchproducer.png)