---
title: Replugin-框架组成结构分析
date: 2018/6/13
tags: 
  - 插件化
  - susion
---

>刚开始看`Replugin`时，还是很容易就会被这个框架弄晕的，接下来主要分析一下这个框架的运行结构。

## 常驻进程 Persistent

在Replugin中存在着许多进程：主进程、常驻进程、自定义进程。不过除主进程外，框架的核心就是Persistent进程。它负责插件的管理。那么先来看一下这个进程是怎么运行起来的吧:

### 进程的创建

>Replugin插件的初始化是在 `RePluginApplication.attachBaseContext()`开始的。这个方法最终会调用` RePlugin.App.attachBaseContext(this, c);`来初始化整个框架。

```
    public static void attachBaseContext(Application app, RePluginConfig config) {

        //配置框架
        sConfig = config;
        sConfig.initDefaults(app);

        //保存好当前进程是常驻进程还是主进程
        IPC.init(app);
        ....
        HostConfigHelper.init();    
        ...
        PMF.init(app);
        PMF.callAttach();

        sAttached = true;
    }
```

>`PMF.init(app);`中，会初始化`PmBase`,并调用`init`方法，下面来看一下这个方法：

```
    void init() {
        if (HostConfigHelper.PERSISTENT_ENABLE) {
            // 默认）“常驻进程”作为插件管理进程，则常驻进程作为Server，其余进程作为Client
            if (IPC.isPersistentProcess()) {
                initForServer(); // 初始化“Server”所做工作
            } else {
                initForClient(); // 连接到Server  -->  A 
            }
        } else {
            if (IPC.isUIProcess()) {    // “UI进程”作为插件管理进程（唯一进程），则UI进程既可以作为Server也可以作为Client
                // 1. 尝试初始化Server所做工作，
                initForServer();
                // 2. 注册该进程信息到“插件管理进程”中
                PMF.sPluginMgr.attach();
            } else {
                initForClient(); // 其它进程？直接连接到Server即可
            }
        }
        ....
    }
```  
>其实第一我们肯定是走到 A 点，因此首先运行的肯定是主进程，因此看一下`initForClient()`:

```
    private final void initForClient() {
        // 1. 先尝试连接
        PluginProcessMain.connectToHostSvc();

        // 2. 然后从常驻进程获取插件列表
        refreshPluginsFromHostSvc();
    }
```

>其实这里链接就是链接到常驻进程，但是常驻进程这个时候肯定是不可能在运行的！PluginProcessMain.connectToHostSvc();在链接过程中就会把常驻进程拉起来，拉起来的方法还是十分有意思的：

```

 static final void connectToHostSvc() {
    Context context = PMF.getApplicationContext();
    IBinder binder = PluginProviderStub.proxyFetchHostBinder(context); //获取常驻进程的binder
 }

private static final IBinder proxyFetchHostBinder(Context context, String selection) {
    Cursor cursor = null;
    try {
        Uri uri = ProcessPitProviderPersist.URI;
        cursor = context.getContentResolver().query(uri, PROJECTION_MAIN, selection, null, null);
        if (cursor == null) {
            return null;
        }
        ....
        IBinder binder = BinderCursor.getBinder(cursor);
        return binder;
    } finally {
        CloseableUtils.closeQuietly(cursor);
    }
}
```

哎呀，看到了Replugin的常驻进程是怎么运行起来的了：*指定一个位于指定进程(ProcessPitProviderPersist)的ContentProvider, 调用query方法，那么这个进程就运行起来了* 。

ps: 这个Provider我在manifest没有直接看到，不过应该是在gradle编译期插入到manifest文件中的。

看一下`ProcessPitProviderPersist`的query方法, 要知道这个query方法现在是运行的常驻进程的binder线程池中的:

```
    @Override
    public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
        sInvoked = true;
        return PluginProviderStub.stubMain(uri, projection, selection, selectionArgs, sortOrder);
    }

    public static final Cursor stubMain(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
        if (SELECTION_MAIN_BINDER.equals(selection)) {
            return BinderCursor.queryBinder(PMF.sPluginMgr.getHostBinder());
        }

        if (SELECTION_MAIN_PREF.equals(selection)) {
            // 需要枷锁否？
            initPref();
            return BinderCursor.queryBinder(sPrefImpl);
        }
        return null;
    }

```

可以看到这里把常驻进程的Binder对象`PmHostSvc`,塞在Cursor中返回给客户端了。因此主进程就获得了常驻进程的Binder，可以和常驻进程进行通信了。

## 常驻进程的作用

>上面我们已经知道`PmHostSvc`就是访问常驻进程的Binder因此常驻进程有什么工程直接看这个类就可以了:

```
class PmHostSvc extends IPluginHost.Stub {

    Context mContext;

    PmBase mPluginMgr;

    //负责Server端的服务调度、提供等工作，是服务的提供方
    PluginServiceServer mServiceMgr;

    //插件管理器。用来控制插件的安装、卸载、获取等
    PluginManagerServer mManager;

    //Receiver不论在哪个进程启动都要经过常驻进程
    PluginReceiverProxy mReceiverProxy;

    .....
}
```

结合我们前面了解的ContentProvider，我们知道ContentProvider的启动也要使用常驻进程的 `PluginProviderClient`。

我看可以围绕常驻进程并结合前面分析画出这样一个图：

![](http://7xrbxa.com1.z0.glb.clouddn.com/galaxywarshipRePluginSimple.png)

