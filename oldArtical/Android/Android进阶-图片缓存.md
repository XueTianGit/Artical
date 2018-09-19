title: Android进阶-缓存
date: 2016/3/3 21:10:18                         
categories: Android
---

# Android进阶-缓存 #

## 引言 ##
- 应该怎么缓存
>- 缓存，无非就是为了加快访问速度，不去做重复的工作，缓存这个主题很复杂
>- 这里先来看一下Android开发中的缓存，应该怎么办
>   - 一般，在Android中开发时，我们调用服务器的接口，一般服务器返回的数据都是以json形式返回的
>   - 进而我们对json解析，进行数据展现
>   - 因此，最方便的就是，直接把服务器返回的json缓存了
>   - 对于图片来说，一般服务器都是返回一个图片的URL，进而去下载图片，进行解析，和展示
>   - 缓存图片时，一般要将从服务器下载的数据缓存了


- 缓存形式
>- 网络缓存（这不能叫缓存了）
>- 本地缓存（保存在SD卡中）
>- 内存缓存（保存在应用运行时的内存中）
>- 缓存顺序： 网络缓存->本地缓存->内存缓存;  数据读取顺序： 内存缓存->本地缓存->网络缓存


## 图片的缓存 ##
下面以对图片的缓存，来探讨一下缓存机制

### 网络缓存 ###
>- 这并不能说是缓存， 因为数据就是从网上来的
>- 那么，主要看一下，怎么从网络上获取图片数据

- AsyncTask
>- 这个类，可以使我们，异步执行任务，当任务结束后，做相应的操作
>- 它其实是Handler+线程池的封装

1. 如何使用
	
		//这3个泛型参数，分别对应下面3个方法的参数（即就是下面3个方法参数的泛型）
		class BitmapTask extends AsyncTask<Object, Void, Bitmap> {

			private ImageView ivPic;
			private String url;
			/**
			 * 后台耗时方法在此执行, 位于子线程
			 */
			@Override
			protected Bitmap doInBackground(Object... params) {
				/*比如，在这里我们可以向网络请求图片*/
				return requestPictureFromNet();  
			}
	
			/**
			 * 更新当前任务执行进度进度, 位于主线程
			 */
			@Override
			protected void onProgressUpdate(Void... values) {
				super.onProgressUpdate(values);
			}
	
			/**
			 * 耗时方法结束后,执行该方法, 位于主线程
			 */
			@Override
			protected void onPostExecute(Bitmap result) {
				//将下载的图片缓存
				LocalCacheUtils.savePicToSD(url, result); //缓存的键为图片的url
				MemoryCacheUtils.savePicToMem(url, result);
		
			}
		}

### 本地缓存 ###
>- 这个其实，并没有什么说的， 就是把图片写到本地
>- 特殊一点的就是，把URL给MD5了

		public class LocalCacheUtils {
			public static final String CACHE_PATH = Environment
					.getExternalStorageDirectory().getAbsolutePath() + "/PicCache";
		
			/**
			 * 从本地sdcard读图片
			 */
			public Bitmap getBitmapFromLocal(String url) {	
					/*....*/
			}
		
			/**
			 * 向sdcard写图片
			 */
			public void setBitmapToLocal(String url, Bitmap bitmap) {
				try {
					String fileName = MD5Encoder.encode(url);  //把URLMD5了，更方便
		
					File file = new File(CACHE_PATH, fileName);
					File parentFile = file.getParentFile();

					if (!parentFile.exists()) {// 如果文件夹不存在, 创建文件夹
						parentFile.mkdirs();
					}
					// 将图片保存在本地
					bitmap.compress(CompressFormat.JPEG, 100,new FileOutputStream(file));
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		}

### 内存缓存 ###
>- 内存缓存是比较麻烦的， 因为各种原因，你必须要防止oom
>- Android系统默认给每个应用分配16MB的内存作为运行空间（不同的手机可能不同）
>- 一般，内存缓存你直接写是很麻烦的，不如直接使用XUtils的BitmapUtils，它的底层已经对图片做了缓存
>- 下面来看一下，如何将图片缓存到本地，而不导致oom

- 使用HashMap
>- 想法很简单， 就是把拿到的图片直接以url为key，以Bitmap对象为值，存到HashMap中
>- 但是，这种做法是极易导致oom的， 原因
>    - java中的引用默认都是强引用
>    - 垃圾回收机制，是不会回收强引用的对象的
>    - 因此，HashMap集合会不但变大，从而导致内存溢出

>- java中的引用类型(依次极易被回收)
>- SoftReference (软引用)
>- PhantomReference(幻引用)
>- WeakRefrence(呵呵)

- 使用HashMap + 弱引用

>- 上面已经看出，使用直接使用HashMap， 由于强引用的原因，易导致oom 
>- 如果，我们将对Bitmap对象的引用该为弱引用的话是不是会好些呢？（让垃圾回收机制可以回收）

	public class MemoryCacheUtils {
	
		private HashMap<String, SoftReference<Bitmap>> mMemoryCache = new HashMap<String, 																	SoftReference<Bitmap>>()；
	
		/**
		 * 从内存读
		 * @param url
		 */
		public Bitmap getBitmapFromMemory(String url) {
			SoftReference<Bitmap> softReference = mMemoryCache.get(url);
			if (softReference != null) {
				Bitmap bitmap = softReference.get();
				return bitmap;
			}
		}
	
		/**
		 * 写内存
		 * @param url
		 * @param bitmap
		 */
		public void setBitmapToMemory(String url, Bitmap bitmap) {
			SoftReference<Bitmap> softReference = new SoftReference<Bitmap>(bitmap);
			mMemoryCache.put(url, softReference);
		}
	}

> - 但是，经过测试，弱引用的效果并不好， 垃圾回收机制回收的太快了
> - 这是因为： 在Android2.3以后，系统会优先将SoftReferece对象给回收了，即使在内存够用的情况下

- LruCache (Least Recentlly Use)
>- 在Android3.0以后， Google推荐使用它来做缓存
>- 他使用最少最近使用算法来解决oom问题，（尽快释放内存）
>- LruCache的用法类似与HashMap
>- 其实LruCahe的底层实现就是使用(LinkedHashMap)来实现的
>- 只不过，他对集合中的对象，进行了一定的算法，使集合不会持久保存对象

	public class MemoryCacheUtils {
		private LruCache<String, Bitmap> mMemoryCache;
	
		public MemoryCacheUtils() {
			long maxMemory = Runtime.getRuntime().maxMemory() ; // 模拟器默认是16M
			//初始化时，需要指定用来做缓存的内存块的大小
			mMemoryCache = new LruCache<String, Bitmap>((int) maxMemory) {
				@Override
				protected int sizeOf(String key, Bitmap value) {
					int byteCount = value.getRowBytes() * value.getHeight();// 获取图片占用内存大小
					return byteCount;
				}
			};
		}
	
		/**
		 * 从内存读
		 * 
		 * @param url
		 */
		public Bitmap getBitmapFromMemory(String url) {
			return mMemoryCache.get(url);
		}
	
		/**
		 * 写内存
		 */
		public void setBitmapToMemory(String url, Bitmap bitmap) {
			mMemoryCache.put(url, bitmap);
		}
	}


- 图片压缩

>- 仅仅使用LruCache， 来进行图片缓存的话，还是可能导致oom
>- 因此，我们应进一步优化缓存内容
>- 我们可以将图片压缩以后再，缓存起来
>- 对于图片的压缩，一般根据图片的比例来进行压缩
>- 图片所占的内存与图片大小无关，只与其所占像素多少有关
>- 图片的压缩主要是使用 BitmapFactroy.Options对象来完成
>- Xutils的源码是这样写的

	BitmapFactory.Options options = new BitmapFactory.Options();
  	/*......*/
   	options.inSampleSize = calculateInSampleSize(options, maxSize.getWidth(), maxSize.getHeight());  //计算压缩比

	 public static int calculateInSampleSize(BitmapFactory.Options options, int maxWidth, int maxHeight) {
	        final int height = options.outHeight;
	        final int width = options.outWidth;
	        int inSampleSize = 1;
	
	        if (width > maxWidth || height > maxHeight) {
	            if (width > height) {
	                inSampleSize = Math.round((float) height / (float) maxHeight);
	            } else {
	                inSampleSize = Math.round((float) width / (float) maxWidth);
	            }
	
	            final float totalPixels = width * height;
	
	            final float maxTotalPixels = maxWidth * maxHeight * 2;
	
	            while (totalPixels / (inSampleSize * inSampleSize) > maxTotalPixels) {
	                inSampleSize++;
	            }
	        }
	        return inSampleSize;
	    }

		
	





