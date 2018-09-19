title: Android中Matrix的使用
date: 2016/11/3 18:27:44       
categories: Android
---

##  学习在Android中画图

> 参考学习项目：http://www.jcodecraeer.com/a/opensource/2016/0812/6531.html
>
> 参考博客：http://blog.csdn.net/cquwentao/article/details/51445269

### 感悟

- 在Android中控制一个图像的大小，形状变化，是通过图像矩阵来操控的：Matrix

- Matrix可以将一个点转换为另一个点

  - Matrix常用的4个方法	

    - Translate 平移变换 , Scale 缩放变换， Rotate 旋转变换，Skew 错切变换。

       默认时，这四种变换都是围绕（0，0）点变换的，当然可以自定义围绕的中心点，通常围绕中心点。

    - 对于这4种方法，分别提供了：

      > set（用于设置Matrix中的值）
      >
      > post（后乘，根据矩阵的原理，相当于左乘）
      >
      > pre（先乘，相当于矩阵中的右乘）
      >
      > - 关于变换的set方法都可以带来不同的效果，但是每个set都会把上个效果清除掉，例如依次调用了setSkew,setTranslate，那么最终只有setTranslate会起作用。
      >
      > - 对于前后乘
      >
      >   ```java
      >   Matrix matrix = new Matrix();
      >   matrix.setTranslate(100, 1000);
      >   matrix.preScale(0.5f, 0.5f);
      >   //最终效果是：  既有缩放， 也有平移
      >
      >   matrix.setTranslate(100, 1000);
      >   matrix.postScale(0.5f, 0.5f);
      >   //最终效果是：  平移的效果也被缩放了
      >   ```

  > 例： 将一个Drawable进行平移和缩放

  ```java
  float offsetX = (getWidth() - drawable.getWidth()) / 2;
  float offsetY = (getHeight() - drawable.getHeight()) / 2;
  drawable.getMatrix().postTranslate(offsetX, offsetY);

  float scaleFactor;
  if (getWidth() < getHeight()) {
    scaleFactor = (float) getWidth() / stickerDrawable.getIntrinsicWidth();
  } else {
    scaleFactor = (float) getHeight() / stickerDrawable.getIntrinsicWidth();
  }
  drawable.getMatrix().postScale(scaleFactor / 2, scaleFactor / 2, getWidth() / 2, getHeight() / 2);
  ```

  >  对于程序中，一个特别有用的方法对是setScale和postTranslate，它们允许跨单个轴(或者两个轴)翻转图像。如果以一个负数缩放，那么会将该图像绘制到坐标系统的负值空间。由于(0,0)点位于左上角，使用x轴上的负数会导致向左绘制图像。因此我们需要使用postTranslate方法，将图像向右移动，如：
  >
  >  matrix.setScale(-1, 1);
  >  matrix.postTranslate(bmp.getWidth(),0);

  - 指定点在Matrix中的位置， 获得点的坐标

    ```java
    float[] fourPeekPointer = new float[]{ 0f, 0f, getWidth(), 0f, 0f, getHeight(), getWidth(), getHeight()};
    float[] dst = new float[8];
    mMatrix.mapPoints(dst, src);
    //获得4个点的坐标，并按照顺序存储在目标数组中
    ```


    //获得矩形
    RectF dst = new RectF();
    mMatrix.mapRect(dst, new RectF(0, 0, getWidth(), getHeight()));
    ​```


- Paint

  ```java
  setAntiAlias();            //设置画笔的锯齿效果
  setColor();                 //设置画笔的颜色
  setARGB();                 //设置画笔的A、R、G、B值
  setAlpha();                 //设置画笔的Alpha值
  setTextSize();             //设置字体的尺寸
  setStyle();                  //设置画笔的风格（空心或实心）
  setStrokeWidth();        //设置空心边框的宽度
  getColor();                  //获取画笔的颜色
  ```


- Canvas 画布类

  ```java
  绘制直线：canvas.drawLine(float startX, float startY, float stopX, float stopY, Paint paint);
  绘制矩形：canvas.drawRect(float left, float top, float right, float bottom, Paint paint);
  绘制圆形：canvas.drawCircle(float cx, float cy, float radius, Paint paint);
  绘制字符：canvas.drawText(String text, float x, float y, Paint paint);
  绘制图形：canvas.drawBirmap(Bitmap bitmap, float left, float top, Paint paint);

   mDrawable.draw(canvas);  //将一个Drawable画到画布上

  canvas.concat(mMatrix); // 可以理解成对matrix的变换应用到canvas上的所有对象
  mDrawable.draw(canvas);  //将矩阵变换，应用到 Drawable

   canvas.drawBitmap(bitma, matrix, paint);  // 将矩阵变换，应用到 Bitmap
  ```



#### Drawable与Bitmap

> Bitmap - 称作位图，一般位图的文件格式后缀为bmp，当然编码器也有很多如RGB565、RGB888。作为一种逐像素的显示对象执行效率高，但是缺点也很明显存储效率低。我们理解为一种存储对象比较好。
>
>
> Drawable - 作为Android平下通用的图形对象，它可以装载常用格式的图像，比如GIF、PNG、JPG，当然也支持BMP，当然还提供一些高级的可视化对象，比如渐变、图形等。