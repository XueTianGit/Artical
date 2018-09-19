##Activity的生命周期和启动模式

###Activity正常生命周期需要注意的点
- onPause中不能做一些太耗时的事情，因为这会影响到新Activity的显示，onPause必须先执行完，新Activity的onResume才会执行
- onResume、onPause 应用还在前台，可与用户交互
- onStart、onStop  应用在后台,这两个方法用来区分Activity是否可见

###Activity异常生命周期需要注意的点
>资源相关的系统配置发生改变导致Activity被杀死并重新创建

在这时，Activity会被销毁，其onPause、onStop、onDestroy均会被调用，同时，由于Activity异常终止，系统会调用onSaveInstanceState来保存当前Activity状态。这个方法的调用时机是在onStop之前，它和onPause没有既定时序关系。
当Activity被重新创建后，系统会调用onRestoreInstanceState，并且把Activity销毁时onSaveInstanceState方法所保存的Bundle对象作为参数同时传递给onRestoreInstanceState和onCreate方法。我们可以去除销毁时保存的数据，来恢复相关状态。

Note：系统只在Activity异常终止的时候才会调用onSaveInstanceState和onRestoreInstanceState来存储和恢复数据。并且onRestoreInstanceState被调用时，器参数bundle必不为null

- 系统对于异常终止状态的恢复

系统会默认为我们保存当前Activity的视图结构，并且在Activity重启后为我们恢复这些数据，比如文本框中用户输入的数据、ListView滚动的位置等。
至于每个View系统能够恢复哪些数据，可以查看View的onSaveInstanceState和onRestoreInstanceState的实现。

>资源内存不足导致低优先级的Activity被杀死

在这种情况下，数据的存储和恢复和上面过程相同，这里主要看一下，Activity被杀死的优先级顺序：

1. 前台Acitivity:正在和用户交互的Activity，优先级最高
2. 可见但非前台Activity:比如Activity中弹出了一个对话框，导致Activity可见但是位于后台无法和用户直接交互。
3. 后台Activity:已经被暂停的Activity，比如执行了onStop，优先级最低。

系统内存不足时，会按照优先级顺序杀死上面Activity，因此，一些后台工作不适合脱离四大组件而独自运行在后台中，这样进程很容易被杀死。因此，我们经常把后台工作放在Service中执行，保证我们的工作进程处于前台中。


###避免Activity重建的 configChanges 

>我们可以在AndroidManifest `<Activity/>` 中配置configChanges使activity在一些系统配置改变时不销毁重建：

android:configChanges:
    orientation : 屏幕旋转时不重建
    keyboardHidden : 键盘显示时不重建
    screenSize : 屏幕尺寸信息发生变化(比如旋转屏幕)不重建
    local : 设备的本地位置发生了改变，一般指切换了系统语言
    
如果在`<Activity/>`标签中指定了上面这些配置，那么放配置发生变化时是不会销毁重建Activity的，取而代之会调用Activity的onConfigurationChange方法。

###Activity的启动模式

1. standard:在这个模式下，谁启动了这个Activity，那么这个Activity就运行在启动它的那个Activity所在的栈中。对于非Activity类型的Context（比如ApplicationContext）我们需要为待启动的Activity指定 FLAG_ACTIVITY_NEW_TASK标记位，这时启动的时候就会为它创建一个新的任务栈。
2. singleTop:栈顶复用模式，如果新的Activity已经位于任务栈的栈顶，那么次Activity不会被重新创建，并且onNewIntent方法会被调用。
3. singleTask:栈内复用模式，只要Activity在一个栈中存在，那么多次启动此Activity都不会创建实例，并会调用onNewIntent方法。同时默认具有clearTop的效果。
4. singleInstance:一个Activity运行在一个栈中。

###Activity启动所需的任务栈

任务栈分为前台任务栈，一般一个运行在前台的应用它的Activity所在的栈即为前台任务栈，而在后台运行的
应用其任务栈为后台任务栈。

在默认情况下，所有Activity所需的任务栈的名字为应用的包名。

TaskAffinity

>这个属性主要和singleTask启动模式或者allowTaskReparenting属性配对使用，在其他情况下没有意义。

- 当TaskAffinity和singleTask启动模式配对使用时，它是具有该模式的Activity的目前任务栈的名字。即，待启动的Activity会运行在名字和TaskAffinity相同的任务栈中。standard则没有这个效果。

- 当TaskAffinity和allowTaskReparenting结合使用时。allowTaskReparenting为true的情况下，如果A应用启动了B应用的C Activity。则C Activity会放到B的Activity任务栈中。因为C Activity的TaskAffinity属性肯定和A的Activity的不同，allowTaskReparenting为true就会新建B的任务栈。并且在这种情况下，启动B会直接看到C Activity而不是B的主界面。其实就是B应用已经启动并创建默认的任务栈了


###Activity的Flags

- FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS
具有这个标记位的Avtivity不会出现在历史Activity的列表中。

- FLAG_ACTIVITY_NEW_TASK
即指定“singleTask”启动模式

- FLAG_ACTIVITY_CLEAR_TASK
清空所有之前的Activity，变成栈中的 root Activity。这个标记为必须和FLAG_ACTIVITY_NEW_TASK一块使用。

>以启动欢迎页为例(这个Activity应为栈中的唯一的Activity)

```
   Intent intent = new Intent(context, WelcomeActivity.class);
   intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK |Intent.FLAG_ACTIVITY_NEW_TASK);  //清除前面所有的Activity
```

###IntentFilter的匹配规则

我们可以通过intent匹配`<intent-filter/>`标签中的 action 、category、data来隐式启动Activity。匹配规则如下：

>只有一个Intent同时匹配一个`<intent-filter/>`中的action类别、category类别、data类别才算完全匹配。

####action的匹配规则
- Intent中的action必须能够和`<intent-filter/>`中的action匹配，即action的字符串完全一样(区分大小写的)
- 一个`<intent-filter/>`可以有多个action，和任何一个匹配成功即算action匹配成功

####category的匹配规则
- Intent中可以没有category
- Intent中如果有category，那么不管有几个，每个都要能够和`<intent-filter/>`中的任何一个category相同
- 对于不添加category的Intent，要想成功启动一个Activity，那么这个Activity必须要在`<intent-filter/>`指定"android.intent.category.DEFAULT"这个category

####data的匹配规则
- 如果过滤规则中定义了data，那么Intent中必须也要定义可匹配的`<intent-filter/>`中的某个data。

data由两部分组成，mimeType和URI，mimeType指媒体类型，比如image/jpeg、audio/mpeg4-generic等。

>URI的 Path、pathPattern和pathPrefix

```
    <data
        android:Path="string"
        android:pathPattern="string"
        android:pathPrefix="string"/>
```

Path表示完整的路径信息。
pathPattern也表示完整的路径信息，但它里面可以包含通配符"*"，代表0或多个任意字符。
pathPrefix表示路径的前缀信息。

>对于mimeType的匹配实例

```
    <intent-filter>
        <data android:mimeType="image/*">
    <intent-filter/>
```

上面这个匹配规则指定媒体类型为所有类型的图片，那么Intent中的mimeType属性必须为'image/*'才能匹配。这种情况下虽然过滤规则中没有指定URI，但是却有默认值，URI的默认值为content和file。即下面：

```
    intent.setDataAndType(Uri.parse("file://abc"), "image/png")
```

####确保隐式启动成功
PackageManager的resolveActivity方法或者Intent的resolveActivity方法，如果找不到匹配的Activity就会返回null。

PackageManager还提供了queryIntentActivities方法，这个方法和resolveActivity方法不同的是：它不是返回最佳匹配的Activity信息而是返回所有成功匹配的Activity信息。

```
    public abstract List<ResolveInfo> queryIntentActivities(Intent intent, int flags);
    
    public abstract ResolveInfo resolveActivity(Intent intent, int flags);
```

flags，应该使用 MATCH_DEFAULT_ONLY 这个标记位，即仅仅匹配那些在intent-filter中生命DEFAULT  
 category的Activity,即只要上述两个方法不返回null，那个startActivity一定可以成功。如果不使用这个标记位，就可以把intent-filter中category不含DEFAULT的那些Activity给匹配出来，从而导致startActivity失败。**因为不含有DEFAULT的那些Activity是无法接收隐式Intent的。**


>总结自《Android开发艺术探索》












