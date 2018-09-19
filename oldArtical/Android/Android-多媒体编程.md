title: Android-多媒体编程
date: 2016/2/29 8:28:18        
categories: Android
---

# Android-多媒体编程 #
> 多媒体编程是指对文字、图片、音频、视频的程序设计编程。
> 这里做简单了解。

## 图片 ##

->  图片的一些基础知识 

1）计算机中图片大小的计算
   图片大小 = 图片的总像素 * 每个像素占用的大小
	依据每个像素可表示的颜色种类，分为：

* 单色位图：只能表示2种颜色
	* 使用两个数字：0和1
	* 使用一个长度为1的二进制数字就可以表示了
	* 每个像素占用1/8个字节
* 16色位图：能表示16种颜色
	* 需要16个数字：0-15，0000 - 1111
	* 使用一个长度为4的二进制数组就可以表示了
	* 每个像素占用1/2个字节
* 256色位图：能表示256种颜色
	* 需要256个数字：0 - 255,0000 0000 - 1111 1111
	* 使用一个长度为8的二进制数字
	* 每个像素占用1个字节
* 24位位图：
	* 每个像素占用24位，也就是3个字节，所在叫24位位图
	* R：0-255,需要一个长度为8的二进制数字，占用1个字节
	* G：0-255,需要一个长度为8的二进制数字，占用1个字节
	* B：0-255,需要一个长度为8的二进制数字，占用1个字节

### 加载（大）图片 ###
>-  平常我们在android应用中显示图片时，可能直接将图片资源设置进ImageView或在xml文件中配置就行了。
>- 但是这样是有问题的， 对于小图片的加载时可以的，不过对于大图片的加载，手机可能会受不了，（当然，现在的手机肯定是OK的）
>- **特别是在模拟器中，如果模拟器的内存小的话，会直接崩掉。**
>- 综上所述，对于大图片的加载，若果直接加载而不作处理的话，可能会非常耗费内存。

> - 为什么会如此耗费内存呢？一般一个图片最大也就MB级别吧？这里以加载24位位图图片为例
> - 这是因为在android中是以ARGB表示每个像素，每个像素占用4个字节，所以占用内存资源挺大的。（A表示透明度）

>- 对于大图片的加载，我们一般会先进行缩放，这里我们以屏幕的尺寸为标尺来进行图片的缩放。

下面是一段简单的图片加载代码：

    	Options opt = new Options();       //Options对象主要用来封装解析图片时图片的参数，
    	opt.inJustDecodeBounds = true;     //由于先进行图片的缩放， 所以先不为像素申请内存，只为获取图片宽高
    	BitmapFactory.decodeFile("sdcard/dog.jpg", opt);  //解析图片
    	//拿到图片宽高
    	int imageWidth = opt.outWidth;
    	int imageHeight = opt.outHeight;
    	
    	Display dp = getWindowManager().getDefaultDisplay();   //Display代表手机屏幕对象
    	//拿到屏幕宽高
		int screenWidth = dp.getWidth();
    	int screenHeight = dp.getHeight();
    	
    	//计算缩放比例
    	int scale = 1;
    	int scaleWidth = imageWidth / screenWidth;  
    	int scaleHeight = imageHeight / screenHeight;
    	if(scaleWidth >= scaleHeight && scaleWidth >= 1){   //，为了适应屏幕显示， 取较大值作为图片的缩放比例
    		scale = scaleWidth;
    	}
    	else if(scaleWidth < scaleHeight && scaleHeight >= 1){
    		scale = scaleHeight;
    	}
    	
    	//设置缩放比例
    	opt.inSampleSize = scale;      //设置解析参数：缩放比例
    	opt.inJustDecodeBounds = false;
    	Bitmap bm = BitmapFactory.decodeFile("sdcard/dog.jpg", opt);   //按缩放比例解析图片
    	
    	ImageView iv = (ImageView) findViewById(R.id.iv);
    	iv.setImageBitmap(bm);


### 对图片内容做修改 ###
> 这里的内容修改指的是对图片的简单处理，比如旋转、平移、镜面等。
> 既然是对图片的绘制， 这里要用到对象有Paint、Canvas（画板）

#### 在内存中创建图片的副本 ####
> 直接加载的Bitmap对象是只读的，无法修改，要修改图片只能在内存中创建出一个一模一样的bitmap副本，然后修改副本。

		//加载原图
		Bitmap srcBm = BitmapFactory.decodeFile("sdcard/photo3.jpg");
        
        //创建与原图大小一致的空白bitmap
        Bitmap copyBm = Bitmap.createBitmap(srcBm.getWidth(), srcBm.getHeight(), srcBm.getConfig());
        //定义画笔
        Paint paint = new Paint();
        //把纸铺在画版上
        Canvas canvas = new Canvas(copyBm);
        //把srcBm的内容绘制在copyBm上
        canvas.drawBitmap(srcBm, new Matrix(), paint);    //矩阵对象俺的传入
        
        iv_copy.setImageBitmap(copyBm);   

#### 简单的图片特效处理 ####

> - 对图片进行特效处理，依赖的是Matrix(还记不记得大学中学过的线性代数， 恶魔)。
> - 哎，不过，对矩阵不熟悉也没关系，google已经考虑到了这一点，已经将一些简单图片矩阵处理的方法封装在了Matrix中。
> - 所以，对图片简单特效的处理，只需调用这些已经封装好的方法就可以了。

* 首先定义一个矩阵对象

		Matrix mt = new Matrix();
* 缩放效果

		//x轴缩放1倍，y轴缩放0.5倍
		mt.setScale(1, 0.5f);

* 旋转效果

		//以copyBm.getWidth() / 2, copyBm.getHeight() / 2点为轴点，顺时旋转30度
		mt.setRotate(30, copyBm.getWidth() / 2, copyBm.getHeight() / 2);
* 平移

		//x轴坐标+10，y轴坐标+20
		mt.setTranslate(10, 20);
* 镜面

		//把X坐标都变成负数
		mt.setScale(-1, 1);
		//图片整体向右移
        mt.postTranslate(copyBm.getWidth(), 0);
* 倒影

		//把Y坐标都变成负数
		mt.setScale(1, -1);
		//图片整体向下移
        mt.postTranslate(0, copyBm.getHeight());

### 简单的图片应用 ###

- 画画板
> 画画板的的实现，其实就是在一张空白的图片上，按照用户的指示进行简单的绘制。

- 核心思想：
>  给ImageView设置触摸侦听，得到用户的触摸事件，并获知用户触摸ImageView的坐标
  记录用户触摸事件的XY坐标，绘制直线

		iv.setOnTouchListener(new OnTouchListener() {
			
			@Override
			public boolean onTouch(View v, MotionEvent event) {
				// TODO Auto-generated method stub
				switch (event.getAction()) {
				//触摸屏幕
				case MotionEvent.ACTION_DOWN:
					//得到触摸屏幕时手指的坐标
					startX = (int) event.getX();
					startY = (int) event.getY();
					break;
				//在屏幕上滑动
				case MotionEvent.ACTION_MOVE:
					//用户滑动手指，坐标不断的改变，获取最新坐标
					int newX = (int) event.getX();
					int newY = (int) event.getY();
					//用上次onTouch方法得到的坐标和本次得到的坐标绘制直线
					canvas.drawLine(startX, startY, newX, newY, paint);
					iv.setImageBitmap(copyBm);
					startX = newX;
					startY = newY;
					break;

				}
				return true;
			}
		});
* 刷子效果，加粗画笔

		paint.setStrokeWidth(8);
* 调色板，改变画笔颜色

		paint.setColor(Color.GREEN);

* 保存图片至SD卡

		FileOutputStream fos = null;
		try {
			fos = new FileOutputStream(new File("sdcard/simple.png"));
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		//保存图片
    	copyBm.compress(CompressFormat.PNG, 100, fos);

> - 在这里我们将图片保存后，并不能在图库中直接看到我们的图片， 这是因为：
> - 系统每次收到SD卡就绪广播时，都会去遍历sd卡的所有文件和文件夹，把遍历到的所有多媒体文件都在MediaStore数据库保存一个索引，这个索引包含多媒体文件的文件名、路径、大小
> - 图库每次打开时，并不会去遍历sd卡获取图片，而是通过内容提供者从MediaStore数据库中获取图片的信息，然后读取该图片。

为了让我们的图片在保存之后，能够直接在图库中看到。我们可以手动发送sd卡就绪广播， 从而让系统去遍历sd卡，从而把我们新绘制的图片索引保存到MediaStore数据库中。

		Intent intent = new Intent();
    	intent.setAction(Intent.ACTION_MEDIA_MOUNTED);
    	intent.setData(Uri.fromFile(Environment.getExternalStorageDirectory()));
    	sendBroadcast(intent);

2）撕衣服
>- 曾经有个很火的撕衣服应用，你懂得。那么它的原理是什么呢？
>- 其实就是准备两张图片，一张是穿衣服的，一张是没穿的。两张图片重叠显示。
>- 用户滑动屏幕时，触摸的是外衣照，把手指经过的像素都置为透明，内衣照就显示出来了。

核心代码：

	 iv.setOnTouchListener(new OnTouchListener() {		
			@Override
			public boolean onTouch(View v, MotionEvent event) {
				switch (event.getAction()) {
				case MotionEvent.ACTION_MOVE:
					int newX = (int) event.getX();
					int newY = (int) event.getY();
					//把指定的像素变成透明
					copyBm.setPixel(newX, newY, Color.TRANSPARENT);
					iv.setImageBitmap(copyBm);
					break;

				}
				return true;
			}
		});


> 当然，每次设置一个像素点为透明色，似乎不够刺激。
> 以触摸的像素为圆心，半径为5画圆，圆内的像素全部置为透明

		for (int i = -5; i < 6; i++) {
			for (int j = -5; j < 6; j++) {
				if(Math.sqrt(i * i + j * j) <= 5)
					copyBm.setPixel(newX + i, newY + j, Color.TRANSPARENT);
			}
		}

> NT： 注意像素点的设置不要超出图片范围，不然程序会崩掉的。

## 音频播放 ##
> Android中对于音乐、视频的处理相关的对象是MediaPlayer。

MediaPlayer的简单使用：

    MediaPlayer mp = new MediaPlayer();  
	mp.reset();    
    mp.setDataSource(PATH_TO_FILE);   
    mp.prepare();   
    mp.start();

> **Android文档中指出， 我们new出来的这个MediaPlayer可能为空，所以我们应该在使用前检查一下是否为空。**

音乐播放的几个点：
- 文件路径不仅可以设置为本地文件， 还可以是网络文件
- mp.prepare()这个方法，一般是不调用的，因为同步准备就意味着，可能要让应用没反应，用户体验不好，

	一般应使用异步准备，并设置准备监听：
	player.prepareAsync();
	player.setOnPreparedListener(new OnPreparedListener() {
		//准备完毕时，此方法调用
	    @Override
		public void onPrepared(MediaPlayer mp) {
			player.start();
			//addTimer()；
		}
	});  

- 当MediaPlayer不使用时，我们应该释放其占用的资源（Java不会释放，因为这是个对象是底层调用C来操作硬件的， 而c语言的资源是需手动释放的）。

	      mp.release();
    	  mp = null;

- MediaPlayer一般在使用前都要先reset一下。

### 简单音乐播放器的实现 ###
> 参考市面上的一些音乐播放器。 音乐播放的代码应该写在服务中，这样才能保证当应用切换到后台后仍然能够继续播放。

> 基本步骤

- 写一个音乐播放服务，里面含有音乐播放的基本方法
- 将供外部调用播放方法暴露成接口，并提供中间人对象（访问服务方法的对象）。

----------

	public IBinder onBind(Intent intent) {
		// TODO Auto-generated method stub
		return new MusicController();    //MusicController调用了音乐服务的音乐相关方法
	}

- 在主activity中，启动服务，拿到中间人对象，并根据用户需求调用播放音乐的方法。

----------

		intent = new Intent(this, MusicService.class);
		startService(intent);     //不与activity共生死，即保证后台调用
		conn = new MyserviceConn();   
		bindService(intent, conn, BIND_AUTO_CREATE);  //为了访问服务的方法

#### 给音乐播放器添加进度条 ####

> 进度条采用SeekBar, 至于进度的改变我们采用定时器来实现
> 查看上面的 player.setOnPreparedListener()方法， 我们在音乐开始播放时，启动了定时器。
下面是定时器的具体代码。

----------

		public void addTimer(){
			if(timer == null){
				timer = new Timer();
				timer.schedule(new TimerTask() {
					
					@Override
					public void run() {
						//获取歌曲总时长
						int duration = player.getDuration();
						//获取歌曲当前播放进度
						int currentPosition= player.getCurrentPosition();
						
						//去主线程中刷新UI
						Message msg = MainActivity.handler.obtainMessage();
						//把进度封装至消息对象中
						Bundle bundle = new Bundle();
						bundle.putInt("duration", duration);
						bundle.putInt("currentPosition", currentPosition);
						msg.setData(bundle);
						MainActivity.handler.sendMessage(msg);
						
					}
					//开始计时任务后的5毫秒，第一次执行run方法，以后每500毫秒执行一次
				}, 5, 500);
			}

> **定时器实际是开辟子线程来运行的，因此在run方法中我们是不可以刷新UI， 必须在主线程中刷新UI。**

	static Handler handler = new Handler(){
		public void handleMessage(android.os.Message msg) {
			Bundle bundle = msg.getData();
			int duration = bundle.getInt("duration");
			int currentPostition = bundle.getInt("currentPosition");
			//刷新进度条的进度
			sb.setMax(duration);
			sb.setProgress(currentPostition);
		}
	};

#### 拖动进度条，改变音乐播放进度 ####
> 很简单，其实，就是设置一个进度条的监听，并调用MediaPlayer.seekTo()方法就OK了。

	sb.setOnSeekBarChangeListener(new OnSeekBarChangeListener() {
			@Override
			public void onStopTrackingTouch(SeekBar seekBar) {
				//根据拖动的进度改变音乐播放进度
				int progress = seekBar.getProgress();
				//改变播放进度
				mi.seekTo(progress);
			}
	}

	然后在服务中MediaPlayer.seekTo(progress);

## 视频播放 ##

### SurfaceView ###
> 就像使用ImageView显示图片一样，播放视频也得有个容器吧， 这个容器就是SurfaceView。

SurfaceView的特点：

- 它采用双缓冲技术，可以满足那些，对视频流畅、实时更新要求较高的场合。
- SurfaceView是重量级组件，因此Android是不会轻易去创建它的，只有在可见是才会去创建它。
- SurfaceView一旦不可见，就会被销毁，一旦可见，就会被创建

> 简单的讲，双缓冲技术，就是在播放视频时，内存中有两个画布，A画布显示至屏幕，B画布在内存中绘制下一帧画面，
> 绘制完毕后B显示至屏幕，A在内存中继续绘制下一帧画面




### 简单视频播放器 ###
> 当然， 播放视频也是使用的MedaiPlayer（看名字，就知道）。
> 与音乐的播放相比，特别之处就在于，要设置视频显示的地方（当然是SurfaceView啦）

	player = new MediaPlayer();
	player.reset();
	player.setDataSource("sdcard/2.3gp");
	player.setDisplay(sh);
	player.prepare();
    player.start();


- 结合SurfaceView特点，我们这款简单音乐播放器应该
    - 每次SurfaceView创建时才去播放视频，销毁时，暂停播放视频。
    - 依据1， 所以我们要监听SurfaceView的状态。
    
----------
    
		final SurfaceHolder sh = sv.getHolder();
		sh.addCallback(new Callback() {
						
			//surfaceView销毁时调用
			@Override
			public void surfaceDestroyed(SurfaceHolder holder) {
				//每次surfaceview销毁时，同时停止播放视频
				if(player != null){
					currentPosition = player.getCurrentPosition();
					player.stop();
					player.release();
					player = null;
				}
				
			}
			//surfaceView创建时调用
			@Override
			public void surfaceCreated(SurfaceHolder holder) {
				//每次surfaceView创建时才去播放视频
				if(player == null){
					player = new MediaPlayer();
					player.reset();
					try {
						player.setDataSource("sdcard/2.3gp");
						player.setDisplay(sh);
						player.prepare();
						player.start();
						player.seekTo(currentPosition);
					} catch (Exception e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					} 
				}
				
			}
			//surfaceView结构改变时调用
			@Override
			public void surfaceChanged(SurfaceHolder holder, int format, int width,
					int height) {
				// TODO Auto-generated method stub
				
			}
		});
		
	}
### VideoView ###
> - VodeoView是Android封装好的视频框架，使用它开发者可以不用自己控制播放与暂停。
> - 即把视频丢给他其他一切，他自己搞定。
> - VideoView = MediaPlayer + SurfaceView

使用步骤：

- 在界面布局文件中定义VideoView组件，或在程序中创建VideoView组件
- 调用VideoView的如下两个方法来加载指定的视频
   setVidePath(String path)：加载path文件代表的视频
   setVideoURI(Uri uri)：加载uri所对应的视频
- 将视频控制交给MediaController

代码范例：

	 protected void onCreate(Bundle savedInstanceState) {  
	        super.onCreate(savedInstanceState);  
	        setContentView(R.layout.activity_main);  
	        video1=(VideoView)findViewById(R.id.video1);  
	        mediaco=new MediaController(this);  
	        File file=new File("/mnt/sdcard/通话录音/1.mp4");  
	        if(file.exists()){  
	            //VideoView与MediaController进行关联  
	            video1.setVideoPath(file.getAbsolutePath());  
	            video1.setMediaController(mediaco);  
	            mediaco.setMediaPlayer(video1);  
	            //让VideiView获取焦点  
	            video1.requestFocus();  
	        }  
	          
	    }  


### Vitamio ###
>- MediaPlayer在播放视频方面是十分差劲的，支持的视频格式非常少（3gp、Base64编码的mp4）。
>- 我们在播放视频时，可以使用Vitamio来代替MediaPlayer.它是封装了FFMPEG的视频播放框架，
>- 并且他提供的API全部是javaAPI，且类似于VideoView的API.
>- Vitamio使用很方便， 并且支持的格式很多。

关于Vitamio使用的几点注意：

- Vitamio的源码是以eclipse的项目方式提供的，因此，我们使用的话，要关联这个项目
- 在使用之前，应先检测硬件是否支持
- 其他类似的框架，还有百度媒体云等
- 应使用Vitamio提供的控件

使用Vitamio提供的控件

    <io.vov.vitamio.widget.VideoView
        android:id="@+id/vv"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        />
简单使用：

	protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			//检测是否支持vitamio
			if (!LibsChecker.checkVitamioLibs(this)) {return;}
			
			VideoView vv = (VideoView) findViewById(R.id.vv);
			
			vv.setVideoPath("sdcard/4.rmvb");
			vv.start();
			
			vv.setMediaController(new MediaController(this));
		}

##摄像头
* 启动系统提供的拍照程序
* 
		//隐式启动系统提供的拍照Activity
		Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
		//设置照片的保存路径
        File file = new File(Environment.getExternalStorageDirectory(), "haha.jpg"); 
        intent.putExtra(MediaStore.EXTRA_OUTPUT, Uri.fromFile(file)); 
        startActivityForResult(intent, 0);
* 启动系统提供的摄像程序

		Intent intent = new Intent(MediaStore.ACTION_VIDEO_CAPTURE);

        File file = new File(Environment.getExternalStorageDirectory(), "haha.3gp"); 
        intent.putExtra(MediaStore.EXTRA_OUTPUT, Uri.fromFile(file)); 
		//设置保存视频文件的质量
        intent.putExtra(MediaStore.EXTRA_VIDEO_QUALITY, 1);
        startActivityForResult(intent, 0);