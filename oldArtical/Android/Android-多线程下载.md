title: Android-多线程下载
date: 2016/2/29 8:10:15     
categories: Android
---

# Android-多线程下载 #

当我们在下载网络资源时，开启多个线程下载会比一条线程下载速度快（一个人能干过一群人？）。
>原理：服务器CPU分配给每条线程的时间片相同，服务器带宽平均分配给每条线程，所以客户端开启的线程越多，就能抢占到更多的服务器资源。

多线程下载的关键点：

- 1) 获取下载文件的大小
- 2）可以请求网络资源任意位置的数据
- 3) 确定每条线程下载多少数据
- 4) 计算每条线程下载数据的开始位置和结束位置
- 5）下载临时文件的创建与使用（临时文件应满足可以随机存取数据）


## 多线程下载代码实现 ##

>- 下面代码可以实现多线程下载和断点续传。并且对开启的每个线程创建了一个线程下载进度文件，以保存线程下载的进度，
>- 不过这些进度文件在下载完成后会删除。


	public class MultiDownload {
	
		static int ThreadCount = 3;
		static int finishedThread = 0;
		//确定下载地址
		static String path = "http://192.168.13.13:8080/QQPlayer.exe";
		public static void main(String[] args) {
			
			//发送get请求，请求这个地址的资源
			try {
				URL url = new URL(path);
				HttpURLConnection conn = (HttpURLConnection) url.openConnection();
				conn.setRequestMethod("GET");
				conn.setConnectTimeout(5000);
				conn.setReadTimeout(5000);
				
				if(conn.getResponseCode() == 200){
					//拿到所请求资源文件的长度
					int length = conn.getContentLength();
					
					File file = new File("QQPlayer.exe");
					//生成临时文件
					RandomAccessFile raf = new RandomAccessFile(file, "rwd");
					//设置临时文件的大小
					raf.setLength(length);
					raf.close();
					//计算出每个线程应该下载多少字节
					int size = length / ThreadCount;
					
					for (int i = 0; i < ThreadCount; i++) {
						//计算线程下载的开始位置和结束位置
						int startIndex = i * size;
						int endIndex = (i + 1) * size - 1;
						//如果是最后一个线程，那么结束位置写死
						if(i == ThreadCount - 1){
							endIndex = length - 1;
						}
						//开启下载线程
						new DownLoadThread(startIndex, endIndex, i).start();
					}
				}
			} catch (Exception e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		
		
	}

	class DownLoadThread extends Thread{
		int startIndex;
		int endIndex;
		int threadId;
		
		public DownLoadThread(int startIndex, int endIndex, int threadId) {
			super();
			this.startIndex = startIndex;
			this.endIndex = endIndex;
			this.threadId = threadId;
		}
	
		@Override
		public void run() {
	
			//再次发送http请求，下载原文件
			try {
				File progressFile = new File(threadId + ".txt");
				//判断进度临时文件是否存在
				if(progressFile.exists()){
					FileInputStream fis = new FileInputStream(progressFile);
					BufferedReader br = new BufferedReader(new InputStreamReader(fis));
					//从进度临时文件中读取出上一次下载的总进度，然后与原本的开始位置相加，得到新的开始位置
					startIndex += Integer.parseInt(br.readLine());
					fis.close();
				}
				System.out.println("线程" + threadId + "的下载区间是：" + startIndex + "---" + endIndex);
				HttpURLConnection conn;
				URL url = new URL(MultiDownload.path);
				conn = (HttpURLConnection) url.openConnection();
				conn.setRequestMethod("GET");
				conn.setConnectTimeout(5000);
				conn.setReadTimeout(5000);
				//设置本次http请求所请求的数据的区间
				conn.setRequestProperty("Range", "bytes=" + startIndex + "-" + endIndex);
				
				//请求部分数据，相应码是206
				if(conn.getResponseCode() == 206){
					//流里此时只有1/3原文件的数据
					InputStream is = conn.getInputStream();
					byte[] b = new byte[1024];
					int len = 0;
					int total = 0;
					//拿到临时文件的输出流
					File file = new File("QQPlayer.exe");
					RandomAccessFile raf = new RandomAccessFile(file, "rwd");
					//把文件的写入位置移动至startIndex
					raf.seek(startIndex);
					while((len = is.read(b)) != -1){
						//每次读取流里数据之后，同步把数据写入临时文件
						raf.write(b, 0, len);
						total += len;
	//					System.out.println("线程" + threadId + "下载了" + total);
						
						//生成一个专门用来记录下载进度的临时文件
						RandomAccessFile progressRaf = new RandomAccessFile(progressFile, "rwd");
						//每次读取流里数据之后，同步把当前线程下载的总进度写入进度临时文件中
						progressRaf.write((total + "").getBytes());
						progressRaf.close();
					}
					System.out.println("线程" + threadId + "下载完毕");
					raf.close();
					
					MultiDownload.finishedThread++;
					synchronized (MultiDownload.path) {
						if(MultiDownload.finishedThread == MultiDownload.ThreadCount){
							for (int i = 0; i < MultiDownload.ThreadCount; i++) {
								File f = new File(i + ".txt");
								f.delete();
							}
							MultiDownload.finishedThread = 0;
						}
					}
					
				}
			} catch (Exception e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}

## 手机多线程代码的实现 ##

关键点：

- 相比上面的多线程下载实现，这里添加了进度条。
- 由于子线程中不能刷新UI，这里使用<ProgressBar/>UI控件 
- 这个进度条是android实现，我们可以把刷新UI的代码写在子线程中。

	public class MainActivity extends Activity {
	
		static int ThreadCount = 3;
		static int finishedThread = 0;
		
		int currentProgress;
		String fileName = "QQPlayer.exe";
		//确定下载地址
		String path = "http://192.168.1.101:8080/" + fileName;
		private ProgressBar pb;
		TextView tv;
		
		Handler handler = new Handler(){
			public void handleMessage(android.os.Message msg) {
				//把变量改成long，在long下运算
				tv.setText((long)pb.getProgress() * 100 / pb.getMax() + "%");
			}
		};
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			pb = (ProgressBar) findViewById(R.id.pb);
			tv = (TextView) findViewById(R.id.tv);
		}

	public void click(View v){
		
		Thread t = new Thread(){
			@Override
			public void run() {
				//发送get请求，请求这个地址的资源
				try {
					URL url = new URL(path);
					HttpURLConnection conn = (HttpURLConnection) url.openConnection();
					conn.setRequestMethod("GET");
					conn.setConnectTimeout(5000);
					conn.setReadTimeout(5000);
					
					if(conn.getResponseCode() == 200){
						//拿到所请求资源文件的长度
						int length = conn.getContentLength();
						
						//设置进度条的最大值就是原文件的总长度
						pb.setMax(length);
						
						File file = new File(Environment.getExternalStorageDirectory(), fileName);
						//生成临时文件
						RandomAccessFile raf = new RandomAccessFile(file, "rwd");
						//设置临时文件的大小
						raf.setLength(length);
						raf.close();
						//计算出每个线程应该下载多少字节
						int size = length / ThreadCount;
						
						for (int i = 0; i < ThreadCount; i++) {
							//计算线程下载的开始位置和结束位置
							int startIndex = i * size;
							int endIndex = (i + 1) * size - 1;
							//如果是最后一个线程，那么结束位置写死
							if(i == ThreadCount - 1){
								endIndex = length - 1;
							}

								new DownLoadThread(startIndex, endIndex, i).start();
							}
						}
					} catch (Exception e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
			};
			t.start();
		}
	
	class DownLoadThread extends Thread{
		int startIndex;
		int endIndex;
		int threadId;
		int currentTotal;
		
		public DownLoadThread(int startIndex, int endIndex, int threadId) {
			super();
			this.startIndex = startIndex;
			this.endIndex = endIndex;
			this.threadId = threadId;
		}

		@Override
		public void run() {
			//再次发送http请求，下载原文件
			try {
				File progressFile = new File(Environment.getExternalStorageDirectory(), threadId + ".txt");
				//判断进度临时文件是否存在
				if(progressFile.exists()){
					FileInputStream fis = new FileInputStream(progressFile);
					BufferedReader br = new BufferedReader(new InputStreamReader(fis));
					//从进度临时文件中读取出上一次下载的总进度，然后与原本的开始位置相加，得到新的开始位置
					int lastProgress = Integer.parseInt(br.readLine());
					currentTotal = Integer.parseInt(br.readLine());
					startIndex += lastProgress;
					
					//把上次下载的进度显示至进度条
					//currentProgress += lastProgress;  //3个线程都加了！！！！！
					currentProgress += currentTotal;
					pb.setProgress(currentProgress);
					
					//发送消息，让主线程刷新文本进度
					handler.sendEmptyMessage(1);
					fis.close();
				}
				System.out.println("线程" + threadId + "的下载区间是：" + startIndex + "---" + endIndex);
				HttpURLConnection conn;
				URL url = new URL(path);
				conn = (HttpURLConnection) url.openConnection();
				conn.setRequestMethod("GET");
				conn.setConnectTimeout(5000);
				conn.setReadTimeout(5000);
				//设置本次http请求所请求的数据的区间
				conn.setRequestProperty("Range", "bytes=" + startIndex + "-" + endIndex);
				
				//请求部分数据，相应码是206
				if(conn.getResponseCode() == 206){
					//流里此时只有1/3原文件的数据
					InputStream is = conn.getInputStream();
					byte[] b = new byte[1024];
					int len = 0;
					int total = 0;
					//拿到临时文件的输出流
					File file = new File(Environment.getExternalStorageDirectory(), fileName);
					RandomAccessFile raf = new RandomAccessFile(file, "rwd");
					//把文件的写入位置移动至startIndex
					raf.seek(startIndex);
					while((len = is.read(b)) != -1){
						//每次读取流里数据之后，同步把数据写入临时文件
						raf.write(b, 0, len);
						total += len;
						currentTotal += len;
						System.out.println("线程" + threadId + "下载了" + total);
						
						//每次读取流里数据之后，把本次读取的数据的长度显示至进度条
						currentProgress += len;
						pb.setProgress(currentProgress);
						//发送消息，让主线程刷新文本进度
						handler.sendEmptyMessage(1);
						
						//生成一个专门用来记录下载进度的临时文件
						RandomAccessFile progressRaf = new RandomAccessFile(progressFile, "rwd");
						//每次读取流里数据之后，同步把当前线程下载的总进度写入进度临时文件中
						progressRaf.write((total + "\r\n"+currentTotal).getBytes());
						progressRaf.close();
					}
					System.out.println("线程" + threadId + "下载完毕-");
					raf.close();
					
					finishedThread++;
					synchronized (path) {
						if(finishedThread == ThreadCount){
							for (int i = 0; i < ThreadCount; i++) {
								File f = new File(Environment.getExternalStorageDirectory(), i + ".txt");
								f.delete();
							}
							finishedThread = 0;
						}
					}
					
				}
			} catch (Exception e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		}
	
		}





