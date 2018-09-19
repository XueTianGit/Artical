title: Android进阶 - FlowLayout
date: 2016/3/3 21:08:56                       
categories: Android
---

# Android进阶 - FlowLayout#

	/** 
	 * @author  Susion 
	 * @date 创建时间：2015-11-29 
	 * 1. 向这个布局中添加的控件，会根据其宽度成行的规则排列
	 * 2. 也可以使用这个控件实现：瀑布流的效果
	 */
	
	public class FlowLayout extends ViewGroup {
		
		private int horizontolSpacing=UiUtils.dip2px(13);  //各个控件之间的水平距离
		private int verticalSpacing=UiUtils.dip2px(13);
		
		private Line currentline;   // 当前的行     
		private int useWidth=0;   // 当前行使用的宽度
		private List<Line> mLines=new ArrayList<FlowLayout.Line>();  //用于记录，当前布局中所有子控件所占的行数
		private int width;
		private int height;
		
		public FlowLayout(Context context) {
			super(context);
		}
		public FlowLayout(Context context, AttributeSet attrs, int defStyle) {
			super(context, attrs, defStyle);
		}
		
		public FlowLayout(Context context, AttributeSet attrs) {
			super(context, attrs);
		}
		
		
		// 测量 当前控件Flowlayout 
		// 父类是有义务测量每个孩子的 
		@Override
		protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
			
			mLines.clear();       //由于onMeasure()可能会被多次调用，一次在每一次测量孩子之前，先把上次测量的结果给清除了
			currentline=null;     //刚开始，测量，当前行肯定为空
			useWidth=0;           //当前行已经被占用的宽度也为空
			int parentWidthMode = MeasureSpec.getMode(widthMeasureSpec);
			int parentHeightMode = MeasureSpec.getMode(heightMeasureSpec);  //  获取当前父容器(Flowlayout)的模式
			
			//在摆放子控件时，子控件与父控件(Flowlayout)具有一定的padding
			width = MeasureSpec.getSize(widthMeasureSpec)-getPaddingLeft()-getPaddingRight();  
			height = MeasureSpec.getSize(heightMeasureSpec)-getPaddingBottom()-getPaddingTop();
			
			
			//下面要对每个孩子进行测量
			int childeWidthMode;
			int childeHeightMode;
			//  为了测量每个孩子 需要指定每个孩子测量规则 , 孩子的测量规则应根据父控件的规则来
			childeWidthMode = parentWidthMode==MeasureSpec.EXACTLY ? MeasureSpec.AT_MOST : parentWidthMode;
			childeHeightMode = parentHeightMode==MeasureSpec.EXACTLY ? MeasureSpec.AT_MOST : parentHeightMode;
			
			//制定孩子的测量规则，(模式)
			int childWidthMeasureSpec = MeasureSpec.makeMeasureSpec(width,  childeWidthMode);   //这两个参数，实际上谁在前，谁在后都无所谓
			int childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(height,  childeHeightMode);
			
			currentline=new Line();// 创建了第一行 
			for(int i=0;i<getChildCount();i++)	{  //遍历所有的孩子，把他们一行一行的放到自己维护的行的集合中，以便以后的布局使用
				View child=getChildAt(i);
	
				// 测量每个孩子
				child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
				int measuredWidth = child.getMeasuredWidth();
				
				useWidth+=measuredWidth;// 让当前行加上使用的长度 
				if(useWidth<=width){
					currentline.addChild(child);  //这时候证明当前的孩子是可以放进当前的行里,放进去
					useWidth+=horizontolSpacing;   //再加上间隔
					if(useWidth>width){   //加上间隔就超行了， 因此要换行
						newLine();					//换行 
					}
				}else{				//换行   
					if(currentline.getChildCount()<1){
						currentline.addChild(child);  // 保证当前行里面最少有一个孩子 ， 不然就一直换行了，直到程序挂掉
					}
					newLine();
				}
				
			}
			
			if(!mLines.contains(currentline)){  //当把所有的孩子安排完之后，可能最后一行，没有被记录到集合中
				mLines.add(currentline);// 添加最后一行
			}
			
			//根据我们一共有多少行，来计算这个FlowLayout的高度
			int  totalheight=0;
			for(Line line:mLines){
				totalheight+=line.getHeight();
			}
			totalheight += verticalSpacing * (mLines.size()-1) + getPaddingTop()+getPaddingBottom();
			//我们这个控件(FlowLayout)的高度和宽度， 如我们自己执行onMeasure, 则必须调用这个方法， 以表示测量完成
			setMeasuredDimension(width+getPaddingLeft()+getPaddingRight(), resolveSize(totalheight, heightMeasureSpec));
		}
		
		//换行时，将当前行记录到集合中，并创建新行，
		private void newLine() {
			mLines.add(currentline);// 记录之前的行
			currentline=new Line(); // 创建新的一行
			useWidth=0;
		}
	
		//带表FlowLayout的一行
		private class Line{
			int height=0; //当前行的高度， 应由一行中，最高的控件来决定
			int lineWidth=0;  
			private List<View> children=new ArrayList<View>();  //这一行中所包含的子控件
			/**
			 * 添加一个孩子
			 * @param child
			 */
			public void addChild(View child) {
				children.add(child);
				if(child.getMeasuredHeight()>height){  //找出一行中，最高的孩子，并用他的高度作为行的高度
					height=child.getMeasuredHeight();   
				}
				lineWidth+=child.getMeasuredWidth();    //累加本行的宽度
			}
			public int getHeight() {
				return height;
			}
			/**
			 * 返回孩子的数量
			 * @return
			 */
			public int getChildCount() {
				return children.size();
			}
			
			//本行， 应负责对本行中的孩子进行布局
			public void layout(int l, int t) {
				lineWidth += horizontolSpacing*(children.size()-1);  //重新调整本行的宽度
				int surplusChild=0;    
				int surplus=width-lineWidth;  //一行剩余的宽度
				
				if(surplus>0){
					surplusChild=surplus/children.size();  //平均分配一下剩余的宽度，并均分到每个孩子身边的间隙上
				}
				
				//对本行的孩子进行布局
				for(int i=0;i<children.size();i++){
					View child=children.get(i);
					//child.layout(l, t, getMeasuredWidth(), getMeasuredHeight());  //就可以实现瀑布流的效果
					child.layout(l, t, l+child.getMeasuredWidth()+surplusChild, t+child.getMeasuredHeight());
					l+=child.getMeasuredWidth()+surplusChild;
					l+=horizontolSpacing;
				}
			}
			
		}
		
		
		// 分配每个孩子的位置 
		@Override
		protected void onLayout(boolean changed, int l, int t, int r, int b) {
			l+=getPaddingLeft();
			t+=getPaddingTop();  //确定开始布局的起点
			
			for(int i=0;i<mLines.size();i++){   //对每一行进行布局
				Line line=mLines.get(i);
				line.layout(l,t);   //交给每一行去分配
				t += line.getHeight()+verticalSpacing;
			}
		}
	
	}
