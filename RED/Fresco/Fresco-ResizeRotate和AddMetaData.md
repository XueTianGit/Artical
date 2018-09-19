---
title: Fresco-ResizeRotata和AddMetaData
date: 2018/6/26
tags: 
  - Fresco
  - susion
---

> 这里还是继续Fresco加载网络图片这个流程继续追踪的，在继续追踪前，再看一遍Fresco网络加载图片的流程:

```
内存获取(只读) -> 线程切换 -> 多路复用 -> 内存获取(监听返回结果缓存到内存) -> Decode -> Resize Rotate -> Add Meta Data -> Encode Memory -> （disk cache） -> (webp tracscode) ->  Net Get
```
>ok, 这一节继续看一下 Resize Rotate 和 Add Meta Data 都干了什么。

##  Resize Rotate

```
/**
 * Resizes and rotates JPEG image according to the EXIF orientation data or a specified rotation
 * angle.
 * <p> If the image is not JPEG, no transformation is applied.
 */
public class ResizeAndRotateProducer implements Producer<EncodedImage>{
  @Override
  public void produceResults(
      final Consumer<EncodedImage> consumer,
      final ProducerContext context) {
    mInputProducer.produceResults(new TransformingConsumer(consumer, context), context);
  }
}
```

从注释可以看出来，这个一步操作是对JPEG操作，不会应用到其他的图片上。对于图片的处理是拿到 `mInputProducer` 的结果进行处理。

>逻辑还是很简单的，当监听拿到图片返回结果，开启后台任务做图片的Resize和Rotate

```
  @Override
  protected void onNewResultImpl(@Nullable EncodedImage newResult, @Status int status) {
    ......
    if (isLast || mProducerContext.isIntermediateResultExpected()) {
      mJobScheduler.scheduleJob();
    }
  }

```

>` mJobScheduler.scheduleJob();`最终会调用`doTransform()`，在这个方法中会根据图片的Resize参数和旋转角度算出`smapleSize`,然后再根据图片的旋转角度调用native方法进行处理。

```
    final int softwareNumerator = getSoftwareNumerator(
        imageRequest,
        encodedImage,
        mResizingEnabled);

    final int downsampleRatio = DownsampleUtil.determineSampleSize(imageRequest, encodedImage);
    final int downsampleNumerator = calculateDownsampleNumerator(downsampleRatio);

    final int numerator;
    if (mUseDownsamplingRatio) {
      numerator = downsampleNumerator;
    } else {
      numerator = softwareNumerator;
    }
    is = encodedImage.getInputStream();
    if (INVERTED_EXIF_ORIENTATIONS.contains(encodedImage.getExifOrientation())) {
      final int exifOrientation =getForceRotatedInvertedExifOrientation(imageRequest.getRotationOptions(), encodedImage);
      JpegTranscoder.transcodeJpegWithExifOrientation(
          is, outputStream, exifOrientation, numerator, DEFAULT_JPEG_QUALITY);
    } else {
      final int rotationAngle =
          getRotationAngle(imageRequest.getRotationOptions(), encodedImage);
      JpegTranscoder.transcodeJpeg(
          is, outputStream, rotationAngle, numerator, DEFAULT_JPEG_QUALITY);
    }
```

## Add Meta Data 

```
  private static class AddImageTransformMetaDataConsumer extends DelegatingConsumer<EncodedImage, EncodedImage> {
    .....
    @Override
    protected void onNewResultImpl(EncodedImage newResult, @Status int status) {
        if (newResult == null) {
          getConsumer().onNewResult(null, status);
          return;
        }
        if (!EncodedImage.isMetaDataAvailable(newResult)) {
          newResult.parseMetaData();
        }
        getConsumer().onNewResult(newResult, status);
    }
  }

  /** Sets the encoded image meta data. */
  public void parseMetaData() {
    final ImageFormat imageFormat = ImageFormatChecker.getImageFormat_WrapIOException(getInputStream());
    mImageFormat = imageFormat;
    // BitmapUtil.decodeDimensions has a bug where it will return 100x100 for some WebPs even though
    // those are not its actual dimensions
    final Pair<Integer, Integer> dimensions;
    if (DefaultImageFormats.isWebpFormat(imageFormat)) {
      dimensions = readWebPImageSize();
    } else {
      dimensions = readImageSize();
    }
    if (imageFormat == DefaultImageFormats.JPEG && mRotationAngle == UNKNOWN_ROTATION_ANGLE) {
      // Load the JPEG rotation angle only if we have the dimensions
      if (dimensions != null) {
        mExifOrientation = JfifUtil.getOrientation(getInputStream());
        mRotationAngle = JfifUtil.getAutoRotateAngleFromOrientation(mExifOrientation);
      }
    } else {
      mRotationAngle = 0;
    }
  }
```

>可以看到就是根据图片格式，解析出图片的大小、方向和角度等MetaData。



