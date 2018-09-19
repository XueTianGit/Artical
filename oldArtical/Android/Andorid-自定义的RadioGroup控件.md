title: Andorid-自定义的RadioGroup控件
date: 2016/7/21 16:28:43
categories: Android
---

# 控件简述
>- 该控件用于用于实现多选1， 多个中一个也不选择。
>- 其中的选择项， 可以随意增加

![](http://7xrbxa.com1.z0.glb.clouddn.com/Android-%E5%A4%9A%E9%80%89%E4%B8%80%E6%8E%A7%E4%BB%B6.PNG)


#核心实现原理
>- 这个控件实际上是使用RecycleView实现的
>- 多选一，多不选等对外状态的改变是可以手工代码控制的

    public static List<String> datasetMediumStatus = new LinkedList<>(Arrays.asList("G", "L", "H"));

 	StaggeredGridLayoutManager mediumLayouManager = new StaggeredGridLayoutManager(3, LinearLayoutManager.VERTICAL);
    lvMediumStatus.setLayoutManager(mediumLayouManager);
    mediumAdapter = new CircleCheckBaseAdapter(this, datasetMediumStatus);
    lvMediumStatus.setAdapter(mediumAdapter);
    mediumAdapter.setOnRecyclerViewItemClickListener(new CircleCheckBaseAdapter.OnRecyclerViewItemClickListener() {
        @Override
        public void onItemClick(TextView textView, String text) {
            List<CircleCheckBaseAdapter.PointTextHolder> pointTextItems = mediumAdapter.getPointTextItems();
            for (CircleCheckBaseAdapter.PointTextHolder holder : pointTextItems) {  //标记状态时， 先将其他状态清除
                holder.setInnerTextViewSelected(false);
            }

            textView.setSelected(true);  //将选中的项 标记为被选中
            mediumStatus = text;
        }
    });


	/**
 	* Created by susion on 2016/5/26.
	*/
	public class CircleCheckBaseAdapter extends RecyclerView.Adapter<CircleCheckBaseAdapter.PointTextHolder>{
	    private List<String> datasource = new ArrayList<>();
	
	    private List<PointTextHolder> pointTextItems = new ArrayList<>();
	    private OnRecyclerViewItemClickListener onRecyclerViewItemClickListener;
	
	    private Activity context;
	    public CircleCheckBaseAdapter(Activity context, List<String> dataSource) {
	        this.context = context;
	        this.datasource = dataSource;
	    }
	
	    @Override
	    public PointTextHolder onCreateViewHolder(ViewGroup parent, int viewType) {
	
	        View view = LayoutInflater.from(context).inflate(R.layout.module_info_item, parent, false);
	        PointTextHolder holder =  new PointTextHolder(view);
	        pointTextItems.add(holder);
	
	        holder.tvTextView.setOnClickListener(new View.OnClickListener() {
	            @Override
	            public void onClick(View v) {
	                if( onRecyclerViewItemClickListener != null){
	                    TextView tempTextView = (TextView) v;
	                    onRecyclerViewItemClickListener.onItemClick(tempTextView, tempTextView.getText().toString());
	                }
	            }
	        });
	        return holder;
	    }
	
	    @Override
	    public void onBindViewHolder(PointTextHolder holder, int position) {
	        holder.setData(position);
	    }
	
	    @Override
	    public int getItemCount() {
	        return datasource.size();
	    }
	
	
	
	    public class PointTextHolder extends RecyclerView.ViewHolder {
	        private TextView tvTextView;
	        public PointTextHolder(View itemView) {
	            super(itemView);
	            tvTextView = (TextView) itemView.findViewById(R.id.tv_text_point_info);
	        }
	
	        public void setData(int position){
	            tvTextView.setText(datasource.get(position));
	        }
	
	        public void setInnerTextViewSelected(boolean state){
	            tvTextView.setSelected(state);
	        }
	
	    }
	
	    public interface OnRecyclerViewItemClickListener {
	        void onItemClick(TextView textView, String text);
	    }
	
	
	    public void setOnRecyclerViewItemClickListener(OnRecyclerViewItemClickListener onRecyclerViewItemClickListener) {
	        this.onRecyclerViewItemClickListener = onRecyclerViewItemClickListener;
	    }
	
	    public List<PointTextHolder> getPointTextItems() {
	        return pointTextItems;
	    }
	
	    public void setItemSelect(int position){
	        pointTextItems.get(position).setInnerTextViewSelected(true);
	    }
	
	
	}


# 对于该控件的数据回显 #
> 必须在布局完成后，才能设置数据状态

	lvMediumStatus.addOnLayoutChangeListener(new View.OnLayoutChangeListener() {
            @Override
            public void onLayoutChange(View v, int left, int top, int right, int bottom, int oldLeft, int oldTop, int oldRight, int oldBottom) {
                String mediumStatus2 = sceneCollectPhotoTask.getMediumStatus();
                if (mediumStatus2.equals("G")) {
                    mediumAdapter.setItemSelect(0);
                }
                if (mediumStatus2.equals("L")) {
                    mediumAdapter.setItemSelect(1);
                }

                if (mediumStatus2.equals("H")) {
                    mediumAdapter.setItemSelect(2);
                }
                mediumStatus = mediumStatus2;

            }
        });