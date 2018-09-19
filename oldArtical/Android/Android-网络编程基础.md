title: Android-网络编程基础
date: 2016/2/29 8:09:29   
categories: Android
---

# Android-网络编程基础 #

>- 关键对象：HttpURLConnection,这个对象就像一个简单的浏览器，使用这个对象我们可以请求服务器的数据。
>- 当我们利用这个对象访问网络时，首先应记得添加权限：

> 访问网络：  <uses-permission android:name="android.permission.INTERNET"/>

##  简单使用 HttpURLConnection ##

例如在这里，我们请求服务器的一个图片资源：
		
    	URL url = new URL(address);
    	//获取连接对象，并没有建立连接
    	HttpURLConnection conn = (HttpURLConnection) url.openConnection();
    	//设置连接和读取超时
    	conn.setConnectTimeout(5000);
    	conn.setReadTimeout(5000);
    	//设置请求方法，注意必须大写
    	conn.setRequestMethod("GET");
    	//建立连接，发送get请求
		//conn.connect();
    	//建立连接，然后获取响应吗，200说明请求成功，我们应根据响应码来进行接下来的操作
    	conn.getResponseCode();

->服务器的图片是以流的形式返回给浏览器的 ,如何操作呢：

    	//拿到服务器返回的输入流
    	InputStream is = conn.getInputStream();
    	//把流里的数据读取出来，并构造成图片
    	Bitmap bm = BitmapFactory.decodeStream(is);
		//接下来，对构造的图片进行操作
> Bitmap是Android中表示位图的对象，我们可以用这个对象来构造图

### 屌屌的开源图片查看器 ###
> 如果，我们想查看网络上的图片，可以使用别人写好的一个应用 SmartImageView（去GitHub上查）。
> 使用这个对象，我们就不需要在写网络交互的代码了，查看网络上的图片，几步就能查看，也不用关心
> 主线程阻塞的问题，该应用会自动把与网络交互的代码，放到子线程中。

例如：

	public void click(View v){
		//下载图片
		//1.确定网址
		String path = "http://192.168.13.13:8080/dd.jpg";
		//2.找到智能图片查看器对象
		SmartImageView siv = (SmartImageView) findViewById(R.id.iv);
		//3.下载并显示图片
		siv.setImageUrl(path);
	}

注意的问题:
> 要想成功查看，我们必须使用这个应用作者自己写的布局控件才能成功。

    <com.loopj.android.image.SmartImageView 
        android:id="@+id/iv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        />

## 网络编程时特殊问题 ##

### 网络交互代码不能写在主线程中 ###
由来：
> - 主线程不能被阻塞
> - 1)在Android中，主线程被阻塞会导致应用不能刷新ui界面，不能响应用户操作，用户体验将非常差
> - 2)主线程阻塞时间过长，系统会抛出ANR异常
> - 3)ANR：Application Not Response；应用无响应
> - 4)任何耗时操作都不可以写在主线程
> - 5)因为网络交互属于耗时操作，如果网速很慢，代码会阻塞，所以网络交互的代码不能运行在主线程

即要这样写

	public void click(View v){
		Thread t = new Thread(){
			@Override
			public void run() {
				//网络交互的代码写在这里、
			}

		};
		t.start();
		
	}

> - 只有主线程能刷新
> - 1)刷新ui的代码只能运行在主线程，运行在子线程是没有任何效果的
> - 2)如果需要在子线程中刷新ui，使用消息队列机制

#### 消息队列 ####

消息队列的机制图：


- Looper一旦发现Message Queue中有消息，就会把消息取出，然后把消息扔给Handler对象，Handler会调用自己的handleMessage方法来处理这条消息
- handleMessage方法运行在主线程

代码使用：
> 主线程创建时，消息队列和轮询器对象就会被创建，但是消息处理器对象，需要使用时，自行创建

		//消息队列
		Handler handler = new Handler(){
			//主线程中有一个消息轮询器looper，不断检测消息队列中是否有新消息，如果发现有新消息，自动调用此方法，注意此方法是在主线程中运行的
			public void handleMessage(android.os.Message msg) {
		
			}
		};

> 在子线程中往消息队列里发消息

		//创建消息对象
		Message msg = new Message();
    	//消息的obj属性可以赋值任何对象，通过这个属性可以携带数据
		msg.obj = new String("可以携带任何对象");   
    	//what属性相当于一个标签，用于区分出不同的消息，从而运行不同的代码
		msg.what = 1;
    	//发送消息
    	handler.sendMessage(msg);


>在主线程中通过switch语句区分不同的消息

		public void handleMessage(android.os.Message msg) {
			switch (msg.what) {
			//如果是1，说明属于请求成功的消息
			case 1:
				ImageView iv = (ImageView) findViewById(R.id.iv);
				Bitmap bm = (Bitmap) msg.obj;
				iv.setImageBitmap(bm);
				break;
			case 2:
				Toast.makeText(MainActivity.this, "请求失败", 0).show();
				break;
			}		
		}


## 发送GET请求 ##

		URL url = new URL(path);
		//获取连接对象
		HttpURLConnection conn = (HttpURLConnection) url.openConnection();
		//设置连接属性
		conn.setRequestMethod("GET");
		conn.setConnectTimeout(5000);
		conn.setReadTimeout(5000);
		//建立连接，获取响应吗
		if(conn.getResponseCode() == 200){
				
		}

###GET方式提交数据
* get方式提交的数据是直接拼接在url的末尾

		final String path = "http://192.168.1.104/Web/servlet/CheckLogin?name=" + name + "&pass=" + pass;
### 使用URLEncoder对URL进行编码 ###
* 浏览器在发送请求携带数据时会对数据进行URL编码，我们写代码时也需要为中文进行URL编码，HttpURLConnection他又不像浏览器这么有才

		String path = "http://192.168.1.104/Web/servlet/CheckLogin?name=" + URLEncoder.encode(name) + "&pass=" + pass;

##POST方式提交数据
* post提交数据是用流写给服务器的
* 协议头中多了两个属性

	1）Content-Type: application/x-www-form-urlencoded，		描述提交的数据的mimetype
	2）Content-Length: 										描述提交的数据的长度

NT：对于post提交的中文数据还是要进行URL编码（有浏览器的时候，浏览器已经帮我们做了）

			//给请求头添加post多出来的两个属性
			String data = "name=" + URLEncoder.encode(name) + "&pass=" + pass;
			conn.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
			conn.setRequestProperty("Content-Length", data.length() + "");

* 设置允许打开post请求的流

		conn.setDoOutput(true);

* 获取连接对象的输出流，往流里写要提交给服务器的数据

		OutputStream os = conn.getOutputStream();
		os.write(data.getBytes());

---



