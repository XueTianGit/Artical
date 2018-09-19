title: Andorid-自定义的图片标记控件
date: 2016/7/21 8:45:55  
categories: Android
---

# 控件简述
>- 该控件用于是LDAR项目Android客户端用于标记管道检测点图片，其最终效果如下：

![](http://7xrbxa.com1.z0.glb.clouddn.com/Android-%E6%A3%80%E6%B5%8B%E7%82%B9%E6%A0%87%E8%AE%B0%E7%95%8C%E9%9D%A2.jpg)

>- 控件可以接受事件
>- 双击插入一个标记点
>- 按住可以进行标记点的拖动
>- 控件分为两个状态： 标记状态 和 查看状态
>- 支持导出标记图片
>- 提供插入监听


#核心实现思路
>- 一个标记点分为两个部分， "大头"和"小头"， 分别记下两头的坐标， 进而画出一个标记点
>- 自定义的ImageView 持有当前所有标记点的集合
>- 在onDraw()， 对所有的标记点进行重绘
>- 在onTouchEvent() 方法中确定标记点， 并更改被拖拽的标记的"大头"或"小头"坐标

>- 相关代码： 


	@Override
    public boolean onTouchEvent(MotionEvent event) {

        if(currentStatus == MARK_STATUS){   //当前状态为标记状态

            final float x = event.getX();
            final float y = event.getY();

            switch (event.getAction()){

                case MotionEvent.ACTION_DOWN:

                    if (GestureJudgeUtils.isDoubleClickEvent()) {  //判断是否在200ms内完成双击事件
                        //插入点
                        currentInsertPointX = x;
                        currentInsertPointY = y;
                        showMoreWindow(this);   //展示收集插入标记点信息的PopupWindow
                    }
                    break;

                case  MotionEvent.ACTION_MOVE:
                    if(5 == isSingleClick){   //保证， 按压接近一秒， 标记点才可以进行拖动处理

                        if(!hasSureDrawPoint){    //首先确定标记点

                            needDrawBeanImg =  MarkUtils.needToMovMarkPointFormImg(x, y, markLayoutBeans);   //通过查看接触点的x, y坐标是否在 "小头部分"
                            if(needDrawBeanImg != null){
                                hasSureDrawPoint = true;

                            }else{
                                //拖拽区域可能在文字部分
                                neddDrawBeanText =  MarkUtils.needToMovMarkPointFormText(x, y, markLayoutBeans); //查看接触点的x, y坐标是否在 "大头部分"
                                if(neddDrawBeanText != null){
                                    hasSureDrawPoint = true;
                                }
                            }
                        }else{

                            if(needDrawBeanImg != null){
                                needDrawBeanImg.setImgMark(MarkUtils.ResetDrawPointFor(x, y, getWidth(), getHeight()));     //确定标记点
                            }else{
                                //拖拽的点肯定在文字部分
                                neddDrawBeanText.setTextMark(MarkUtils.ResetDrawPointFor(x, y, getWidth(), getHeight()));
                            }

                            invalidate();  //重绘界面
                        }
                    }else{
                        isSingleClick++;
                    }

                    break;

                case  MotionEvent.ACTION_UP:
                    //清除有关拖拽的状态
                    hasSureDrawPoint = false;
                    needDrawBeanImg = null;
                    neddDrawBeanText = null;
                    isSingleClick = 0;
                    break;
            }

        }
        return true;
    }


  	@Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        invalidatePoint(canvas);
    }

    private void invalidatePoint(Canvas canvas) {

        if(currentStatus == CHECK_STATUS){
            for(int i=0; i<markLayoutBeans.size(); i++){
                LayoutBean bean = markLayoutBeans.get(i);
                MarkUtils.drawMarkLayoutBean(canvas, i, bean, "", currentStatus, false);
            }

            if(currentCheckBeanPosition != -1){
                boolean isCheckPoint = true;
                MarkUtils.drawMarkLayoutBean(canvas, currentCheckBeanPosition, markLayoutBeans.get(currentCheckBeanPosition), "", currentStatus, isCheckPoint);
            }

        }else{  //标记状态
            for(int i=0; i<markLayoutBeans.size(); i++){
                LayoutBean bean = markLayoutBeans.get(i);
                if(sceneCollectModuleTasks.size() > 0){
                    MarkUtils.drawMarkLayoutBean(canvas, i, bean, sceneCollectModuleTasks.get(i).getType(), currentStatus, false);
                }
            }
        }
    }



>- 如何实现重绘

	needDrawBeanImg 或 neddDrawBeanText 成员变量 是当前被拖动标记点的引用，通过重置他们的位置， 进而达到更改标记信息的目的，进而在绘制标记点时达到了重绘的目标

>- 查看状态和标记状态的区分
	
	区分点就是，确定拖拽这个动作， 仅在标记状态下进行。


#相关工具类

>- 屌屌的判断点击次数的方法。
public class GestureJudgeUtils {

    public static long[] mHits = new long[2];// 数组长度表示要点击的次数

    public static boolean isDoubleClickEvent(){
        System.arraycopy(mHits, 1, mHits, 0, mHits.length - 1);
        mHits[mHits.length - 1] = SystemClock.uptimeMillis();//  SystemClock.uptimeMillis()： 开机后开始计算的时间
        return mHits[0] >= (SystemClock.uptimeMillis() - 200);
    }

}
