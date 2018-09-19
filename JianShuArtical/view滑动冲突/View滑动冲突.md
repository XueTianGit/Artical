##View滑动冲突

>如何解决滑动冲突，这里有两种方法
    
- 外部拦截法

简单来说：点击事件都先经过父容器的拦截处理，如果父容器需要此事件就拦截，如果不需要此事件就不拦截。
伪代码如下：

```
    public boolean onInterceptTouchEvent(MotionEvent event){
        boolean intercepted = false;
        int x = (int)event.getX();
        int y = (int)event.getY();
        
        switch(event.getAction()){
            case MotionEvent.ACTION_DOWN:
                intercepted = true;
                break;
            case MotionEvent.ACTION_MOVE:
                if(need) intercepted = true; 
                else intercepted = false;
                break;
            case MotionEvent.ACTION_UP:
                intercepted = false;
                break;
            default:
                break;
        }
        
        return intercepted;
    }
    
    
```

- 内部拦截法

指父容器不拦截任何事件，所有的事件都传递给子元素，如果子元素需要此事件就直接消耗掉，否则就交由父容器进行处理。

```
    //子元素
    public boolean dispatchTouchEvent(MotionEvent event){
        int x = (int)event.getX();
        int y = (int)event.getY();
        
        switch(event.getAction()){
            case MotionEvent.ACTION_DOWN:
                parent.requestDisallowInterceptTouchEvent(true);
                break;
            case MotionEvent.ACTION_MOVE:
                int deltaX = x - mLastX;
                int deltaY = y - mLastY;
                if(!need){
                    parent.requestDisallowInterceptTouchEvent(false);
                }
                break;
            case MotionEvent.ACTION_UP:
                break;
            default:
                break;
        }
        mLastX = x;
        mLastY = y;    
        return super.dispatchTouchEvent(event);
    }
```

>不过内部拦截法有一个前提：父元素要默认拦截除了ACTION_DOWN以外的其他事件，这样当子元素调用parent.requestDisallowInterceptTouchEvent(false)方法时，父元素才能继续拦截所需事件。

>父元素不能拦截ACTION_DOWN的事件的原因是：ACTION_DOWN不受FLAG_DISALLOW_INTERCEPT这个标记位的控制，所以一旦父View拦截ACTION_DOWN事件，那么所有的事件都无法传递到子元素中去。

总结自：《Android开发艺术探索》







