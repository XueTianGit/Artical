##Android常见异常

> ConcurrentModificationException

遍历一个集合的时候不能删除其中的元素。这个异常也可能在多线程操作同一个集合时发生。对于多线程操作可以使用`CopyOnWriteArrayList`

> 代码混淆导致的类找不到的问题 ClassNotFoundException

对于代码中有通过反射使用的类是不能混淆的，比如`Class.forName("class1")`.因此对于一些类不能进行混淆。应该在混淆文件中keep住。当然最常见的混淆类就是 gson entites。

>Adapter的数据源发生变化，但是没有通知View 

我们在使用`RecyclerView`或者`ViewPager`时，应该先加载数据，再设置Adapter。
还有就是在数据源发生变化时，应该立刻调用`notifyDatasetChanged`方法。

> 同时刷新RecyclerView数据导致的角标越界

当我们的列表在进行上拉加载更多时，这时候就不要去触发上拉刷新。这是因为，在加载更多时`RecycelerView`可能已经知道要 count为多少，可是刷新的触发导致数据清空，因此就会角标越界了

>窗体相关的异常

比如像Dialog这种窗体在显示的时候它所依赖的Activity已经销毁了。杜绝此类异常可以这样处理：

1. 在show Dialog时，判读当前的Activity是否finish或者destroy。
2. 在Activity Destroy时，如果Dialog还在show，那么应该dismiss掉它。
3. 不要异步操作对话框，不要在非UI现场 中使用对话框创建、显示和取消。
4. 对话框要在Activity的可控制范围之内和声明周期之内。

不要将非Activity context传递给Dialog。
在使用WindowManager自定义弹出框时不要忘记设置相关权限。

>The specified child already has a parent

当我们在使用一个已经有父View的View作为其它父View的子View时，应先解除它的父子关系。即一个View在视图层级里面只能有一个父View。

>不能在子线程中操作AlertDialg和Toast

`Can't create handler inside thread that has not called Looper.prepare()`

如果你真的想这么干，那么应该准备好Looper
```
    Looper.prepare()
    ....
    Looper.loop()
```

>Resources$NotFoundException

最常见的就是 textView.setText(number)

即没有给这个方法传递string参数，而是传递类int，这是因为这个方法的int重载接收的是一个resId，但是你却给了一个非resId。因此，我们在使用这种方法时应注意。

> UnstatisFiledLinkError

遇到这个Crash，肯定是so格式的文件没有加载到。因此，要去查看各个平台下的对应的so文件是否存在

>TransactionTooLargeException

Binder最大通常限制为1MB，如果大于1MB就会抛出这个异常。通常会发生这个异常的情形是将大图片在Binder中传输。











