title: Andorid-TimeButton
date: 7/22/2016 2:52:04 PM  
categories: Android
---


#控件需求
- 点击控件，显示一个倒计时读秒
- 倒计时过程中，不能对控件做任何操作
- 当倒计时完成后，点击控件，可以输入信息
- 输入信息完毕，控件上显示输入的信息


#控件实现

##控件的布局
- 父布局为FrameLayout
	- 子布局一： 倒计时样式
	- 子布局二： 正常按钮样式

##控件的实现代码
- 主要利用 Handler + Timer + 控件的可见与不可见
	- Handler： Timer的任务每隔一秒调用一次，任务完成后使用Handler完成界面的刷新
	- Timer: 倒计时 使用
	- 控件的可见与不可见： INVISIBLE 与 VISIBLE
	
- 外部可以控制按钮的倒计时时间
- 外部可以控制按钮是否启用功能
- 外部可以获得用户输入的信息
	
>代码实现：

	/**
	 * Created by susion on 2016/5/31.
	 */
	public class TimeButton extends FrameLayout {
	
	    private static final int DESC_TIME = 1;
	    private static final int ENABLE_RL = 2;

	    private Context mContext;
	    private Button startButton;
	    private RelativeLayout rlTimeRoot;
	    private TextView tvTime;
	    private int minCanClickTime = 20; 
	    private int currentCaculateTempTime = minCanClickTime;  //默认倒计时时间
	    private Timer timer;
	    private TimerTask caculateTask;  //倒计时结束后需要做的任务
	    private String readNumber;
	    private long startTime;
	    private long endTime;
	
	    private int useTotalTime;
	
	    private TimeButtonListener timeButtonListener;
	
	    private Handler mTimeHandler = new Handler() {
	        @Override
	        public void handleMessage(Message msg) {
	            if(msg.what == DESC_TIME){
	                tvTime.setText(currentCaculateTempTime + "s");
	            }
	
	            if(msg.what == ENABLE_RL){
	                rlTimeRoot.setEnabled(true);
	                timer.cancel();
	            }
	
	        }
	    };
	
	    public TimeButton(Context context) {
	        super(context);
	        this.mContext = context;
	        initView();
	        initEvent();
	    }
	
	    public TimeButton(Context context, AttributeSet attrs) {
	        super(context, attrs);
	        this.mContext = context;
	        initView();
	        initEvent();
	    }
	
	    private void initView() {
	        View.inflate(mContext, R.layout.time_button, this);
	        startButton = (Button) findViewById(R.id.time_button_button);
	        rlTimeRoot = (RelativeLayout) findViewById(R.id.time_button_time_background);
	        tvTime = (TextView) findViewById(R.id.time_button_time);
	    }
	
	    private void initEvent() {
	
	        startButton.setOnClickListener(new OnClickListener() {
	            @Override
	            public void onClick(View v) {
	
	                if(timeButtonListener != null){
	                    if(!timeButtonListener.onClickButton()){
	                        return;
	                    }
	                }
	
	                startTime = System.currentTimeMillis();
	
	                startButton.setVisibility(INVISIBLE);
	                rlTimeRoot.setVisibility(VISIBLE);
	                timer = new Timer();
	
	                caculateTask = new TimerTask() {
	                    @Override
	                    public void run() {
	                        currentCaculateTempTime--;
	                        mTimeHandler.sendEmptyMessage(DESC_TIME);
	                        if (currentCaculateTempTime > 0) {
	                            rlTimeRoot.setEnabled(false);
	                        } else {
	                            mTimeHandler.sendEmptyMessage(ENABLE_RL);
	                        }
	                    }
	                };
	
	                timer.schedule(caculateTask, 0, 1000);  //1秒调用一次
	            }
	        });
	
	
	        rlTimeRoot.setOnClickListener(new OnClickListener() {
	            @Override
	            public void onClick(View v) {
	
	                endTime = System.currentTimeMillis();
	                useTotalTime = (int) ((endTime - startTime) / 1000);
	
	                new MaterialDialog.Builder(mContext)
	                        .title("请输入读数")
	                        .inputType(InputType.TYPE_CLASS_NUMBER)
	                        .onNegative(new MaterialDialog.SingleButtonCallback() {
	                            @Override
	                            public void onClick(@NonNull MaterialDialog dialog, @NonNull DialogAction which) {
	
	                                startButton.setVisibility(VISIBLE);
	                                rlTimeRoot.setVisibility(INVISIBLE);
	                                currentCaculateTempTime = minCanClickTime;
	
	
	                            }
	                        })
	                        .input(null, null, new MaterialDialog.InputCallback() {
	
	                            @Override
	                            public void onInput(MaterialDialog dialog, CharSequence input) {
	
	                                startButton.setVisibility(VISIBLE);
	                                rlTimeRoot.setVisibility(INVISIBLE);
	
	                                if(!StringUtils.notIsBlankAndEmpty(input.toString())){
	                                    Toast.makeText(mContext, "输入不能为空", Toast.LENGTH_SHORT).show();
	                                    return;
	                                }
	
	                                readNumber = input.toString();
	
	                                if(timeButtonListener != null){
	                                    timeButtonListener.OnInputFinishListener(readNumber);
	                                }
	
	                                currentCaculateTempTime = minCanClickTime;   //清空状态
	                            }
	
	                        }).positiveText("确定").negativeText("取消").show();
	
	
	            }
	        });
	
	    }
	
	    public void setTimeButtonText(String text){
	        startButton.setText(text);
	    }
	
	    public void stopTimer(){
	        timer.cancel();
	    }
	
	
	    public int getMinCanClickTime() {
	        return minCanClickTime;
	    }
	
	    public void setMinCanClickTime(int minCanClickTime) {
	        this.minCanClickTime = minCanClickTime;
	        currentCaculateTempTime = minCanClickTime;
	    }
	
	
	    public interface TimeButtonListener{
	        boolean onClickButton();
	        void OnInputFinishListener(String inputValue);
	    }
	
	    public void setTimeButtonListener(TimeButtonListener timeButtonListener) {
	        this.timeButtonListener = timeButtonListener;
	
	    }
	
	    public long getStartTime() {
	        return startTime;
	    }
	
	    public void setStartTime(long startTime) {
	        this.startTime = startTime;
	    }
	
	    public long getEndTime() {
	        return endTime;
	    }
	
	    public void setEndTime(long endTime) {
	        this.endTime = endTime;
	    }
	}
