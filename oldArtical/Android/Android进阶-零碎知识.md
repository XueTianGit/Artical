title: Android项目-零碎知识
date: 2016/2/28 10:31:15 
categories: Android
---

# Android项目-零碎知识 #

- Point1（两张图片合成一个动画）
	- 使用FrameLayout，将两张图片叠加
	- 对一张图片做动画效果

- Point2(从assets目录拷贝数据到程序中)
	- 在开发是，我们通常将类似数据库这样的文件放在assets目录下， 当应用程序被安装时，为了使这些数据库文件可以被应用正常使用
	- 因此需要把assets目录下的文件，拷贝到应用程序中。因此会有类似下面的代码（以数据库文件为例）
	- 这些工作一般会在闪屏界面进行处理

----------

	/**
	 * 拷贝数据库
	 */
	private void copyDB(String dbName) {
		//拷贝到应用的file目录下   /data/data/appname/file
		File destFile = new File(getFilesDir(), dbName);// 要拷贝的目标地址
		if (destFile.exists()) {
			System.out.println("数据库" + dbName + "已存在!");
			return;
		}

		FileOutputStream out = null;
		InputStream in = null;
		try {
			in = getAssets().open(dbName);
			out = new FileOutputStream(destFile);

			int len = 0;
			byte[] buffer = new byte[1024];

			while ((len = in.read(buffer)) != -1) {
				out.write(buffer, 0, len);
			}

		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			try {
				in.close();
				out.close();
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}

- Point3 (PackageInfo.applicationInfo.sourceDir )
	- 这个sourceDir其实是.apk文件， 例如：  /system/app/DrmProvider.apk

- Point4 （更新应用数据库）
	- 我们应用的数据库，有时需要更新，例如一个手机卫士的病毒数据库
	- 数据的传递可以以json格式
	- 接收到json格式数据后，解析并持久化到数据库
	- json的解析可以使用google的gjson
	- gjson非常好用，一行搞定（将数据封装到对象中）:InfoObject ob = gson.fromJson(jsonStr, InfoObject.class);

- ScrollView的滑动
	- 我们可以做出，当ScrollView包裹的数据超出屏幕时，让他自动滑动，以展现数据的效果
	- 下面post方法，Causes the Runnable to be added to the message queue. The runnable will be run on the user interface thread

----------

           //自动滚动
            slResultView.post(new Runnable() {
                @Override
                public void run() {
                    //一直往下面进行滚动
                    slResultView.fullScroll(slResultView.FOCUS_DOWN);

                }
            });




