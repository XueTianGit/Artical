title: Android进阶-双击&多击事件
date: 2016/3/1 19:14:18     
categories: Android
---

# Android进阶-双击&多击事件 #

双击事件，我们可以很容易的想到怎么做： 可以根据两个单击事件之间的时间间隔来确定多击事件。
例如下面代码：

	public class MainActivity extends Activity {
	
		private long firstClickTime;
	
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
		}
	
		public void onClick(View view) {
			if (firstClickTime > 0) {// 发现之前点击过一次
				if (System.currentTimeMillis() - firstClickTime < 500) {// 判断两次点击是否小于500毫秒
					Toast.makeText(this, "双击啦!", Toast.LENGTH_SHORT).show();
					firstClickTime = 0;//重置时间, 重新开始
					return;
				}
			}
	
			firstClickTime = System.currentTimeMillis();
		}
	}

但是多击事件呢？ 可能有很多的做法， 下面来看一下Google大牛，写的多击事件的代码。（击多少下由你定义）

	public class DragViewActivity extends Activity {

		private ImageView ivDrag;
		/*......*/

		long[] mHits = new long[10];// 数组长度表示要点击的次数
		ivDrag.setOnClickListener(new OnClickListener() {
			@Override
			public void onClick(View v) {
				System.arraycopy(mHits, 1, mHits, 0, mHits.length - 1);   
				mHits[mHits.length - 1] = SystemClock.uptimeMillis();//  SystemClock.uptimeMillis()： 开机后开始计算的时间
				if (mHits[0] >= (SystemClock.uptimeMillis() - 500)) {  //500ms完成10次单击
						/* do something*/
				}
			}
		});
	}

分析一下这段代码：
> 核心思想就是：把每次单击的时间记录在数组中。数组的最后一个元素与第一个元素之差，分别是第一次单击和最后一次单击的时间戳。
> 理所当然，最后一个时间戳减去第一个时间戳，只要满足我们规定的时间限制，就算完成这个多击事件。->  500 >= (SystemClock.uptimeMillis() - mHits[0] 
> 核心： 上面的mHits数组就像一个队列容器， 时间戳就这样流过这个队列



> 从上面的代码可以看出：
> 
- Google大神真nb有没有？（真是另一个层面的程序员了！！）
- 想编出nb的代码数学得学好有没有！！！！！！（你的逻辑思维强吗？）
- 佩服的不行不行的