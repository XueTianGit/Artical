title: Android进阶-线程池
date: 2016/3/3 21:04:36                 
categories: Android
---

# Android进阶-线程池 #
>- 在android中，系统对线程的支持并完美
>- 因此，在我们的程序中并不建议打开多条线程
>- 一般在一个应用中，线程的最佳数量为： cpu核数 * 2 + 1
>- 为了，使我们的应用程序满足系统要求，我们可以使用线程池，来对应用中的线程进行管理

- 线程池的原理
>- 持有固定数量的线程
>- 维护着一个Runnable集合，这可集合中都是Runnable
>- 线程池所管理的线程，不断执行Runnable集合中的任务

原理实现代码：

	public class ThreadPool {
		int maxCount = 3;
		AtomicInteger count =new AtomicInteger(0);// 当前开的线程数  count=0
		LinkedList<Runnable> runnables = new LinkedList<Runnable>();
	
		public void execute(Runnable runnable) {
			runnables.add(runnable);
			if(count.incrementAndGet() <= 3){
				createThread();// 最大开三个线程
			}
		}
		private void createThread() {
			new Thread() {
				@Override
				public void run() {
					super.run();
					while (true) {
						// 取出来一个异步任务
						if (runnables.size() > 0) {
							Runnable remove = runnables.remove(0); //在集合中移除第一个对象 返回值正好是移除的对象
							if (remove != null) {
								remove.run();
							}
						}else{
							//  等待状态   wake();
						}
					}
				}
			}.start();
		}
	}


- 线程池管理者
>- 线程池的原理不难， 但是实现时要考虑的同步，并发等问题太多
>- 在Android系统中，已经给我们提供了一个线程池： ThreadPoolExecutor
>- 下面这个ThreadManager管理这两个线程池

	/*
	 *  1. ThreadManager是单例的
	 *  2. 拥有两个线程池，一个线程池中的线程用于处理比较耗时的操作（比如，联网请求）
	 *                   一个线程池中的线程用于处理相对耗时较少的操作（比如，文件的读取）
	 */
	public class ThreadManager {
			private ThreadManager() {
			}
	
		private static ThreadManager instance = new ThreadManager();
		private ThreadPoolProxy longPool;
		private ThreadPoolProxy shortPool;
	
		public static ThreadManager getInstance() {
			return instance;
		}
	
		// 联网比较耗时
		public synchronized ThreadPoolProxy createLongPool() {
			if (longPool == null) {
				longPool = new ThreadPoolProxy(5, 5, 5000L);
			}
			return longPool;
		}
	
		// 操作本地文件
		public synchronized ThreadPoolProxy createShortPool() {
			if(shortPool==null){
				shortPool = new ThreadPoolProxy(3, 3, 5000L);
			}
			return shortPool;
		}
	
		//线程池的代理
		public class ThreadPoolProxy {
			private ThreadPoolExecutor pool;
			private int corePoolSize;
			private int maximumPoolSize;
			private long time;
	
			public ThreadPoolProxy(int corePoolSize, int maximumPoolSize, long time) {
				this.corePoolSize = corePoolSize;
				this.maximumPoolSize = maximumPoolSize;
				this.time = time;
	
			}
			/**
			 * 执行任务
			 * @param runnable
			 */
			public void execute(Runnable runnable) {
				if (pool == null) {
					// 创建线程池
					/*
					 * 1. 线程池里面管理多少个线程   2. 如果排队满了, 额外的开的线程数   3. 如果线程池没有要执行的任务 存活多久
					 * 4.存活多久的， 时间的单位
					 *  5 如果 线程池里管理的线程都已经用了,剩下的任务 临时存到LinkedBlockingQueue对象中 排队
					 */
					pool = new ThreadPoolExecutor(corePoolSize, maximumPoolSize,
							time, TimeUnit.MILLISECONDS,
							new LinkedBlockingQueue<Runnable>(10));
				}
				pool.execute(runnable); // 调用线程池 执行异步任务
			}

			/**
			 * 取消任务
			 * @param runnable
			 */
			public void cancel(Runnable runnable) {
				if (pool != null && !pool.isShutdown() && !pool.isTerminated()) {
					pool.remove(runnable); // 取消异步任务
				}
			}
		}
	}
