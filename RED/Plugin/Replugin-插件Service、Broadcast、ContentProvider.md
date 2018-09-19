---
title: Replugin-插件Service、Broadcast、ContentProvider
date: 2018/6/13
tags: 
  - 插件化
  - susion
---

## Service

上一节我们了解到插件Activity的Context已经被替换为PluginContext，并且在Android中启动Service只能通过Context对象的startService和bindService函数，因此Replugin在PluginContext中重写了这两个函数，以调用自己设计的启动流程。

```
    @Override
    public ComponentName startService(Intent service) {
        //..
        try {
            return PluginServiceClient.startService(this, service, true);
        } catch (PluginClientHelper.ShouldCallSystem e) {
            return super.startService(service); // 若打开插件出错，则直接走系统逻辑
        } finally {
            if (mContextInjector != null) {
                mContextInjector.startServiceAfter(service);
            }
        }
    }
```
即调用了`PluginServiceClient`的`startService`方法:

```
    public static ComponentName startService(Context context, Intent intent, boolean throwOnFail) {
        // 从 Intent 中获取 ComponentName
        ComponentName cn = getServiceComponentFromIntent(context, intent);
        ....
        intent.setComponent(cn);
        ...
        IPluginServiceServer pss = sServerFetcher.fetchByProcess(process);
        ...
        return pss.startService(intent, sClientMessenger);
        ...
    }
```
上面方法只是保留的大致流程， 将启动的service的intent转化为`ConponentName`,使用`sServerFetcher`获取指定进程的`IPluginServiceServer`，使用`IPluginServiceServer`来启动服务。

```
  public IPluginServiceServer fetchByProcess(int process) {
        ....
        if (process == IPluginManager.PROCESS_PERSIST) {
            IPluginHost ph = PluginProcessMain.getPluginHost();
            pss = ph.fetchServiceServer();
        } else {
            PluginBinderInfo pbi = new PluginBinderInfo(PluginBinderInfo.NONE_REQUEST);
            IPluginClient pc = MP.startPluginProcess(null, process, pbi);
            pss = pc.fetchServiceServer();
        }
        ....
        return pss;
    }
```

对于如何获取指定进程的`IPluginServiceServer`,这里不仔细分析，不过这里可以看到，如果要启动的service位于`PROCESS_PERSIST`进程，则直接获取`PROCESS_PERSIST`进程的`PluginServiceServer.Stub`对象，是的，是一个*Binder*。

因此接下来对于Service的启动就来到了`PROCESS_PERSIST`进程中：

```
    // 启动插件Service。说明见PluginServiceClient的定义
    ComponentName startServiceLocked(Intent intent, Messenger client) {
        intent = cloneIntentLocked(intent);
        ComponentName cn = intent.getComponent();

        final ServiceRecord sr = retrieveServiceLocked(intent);
        .....

        if (!installServiceIfNeededLocked(sr)) {
            return null;
        }

        sr.startRequested = true;

        // 加入到列表中，统一管理
        mServicesByName.put(cn, sr);
        // 从binder线程post到ui线程，去执行Service的onStartCommand操作
        Message message = mHandler.obtainMessage(WHAT_ON_START_COMMAND);
        Bundle data = new Bundle();
        data.putParcelable("intent", intent);
        message.setData(data);
        message.obj = sr;
        mHandler.sendMessage(message);
        return cn;
    }
```

上面代码可以看出，对于要启动的Service，创建了`ServiceRecord`对象。然后调用`installServiceIfNeededLocked`启动，然后保存`ServiceRecord`,然后利用`Handler`来用调用Service的`onStartCommand`方法。

>对于`installServiceIfNeededLocked()`,它内部把线程切换到主线程，然后执行`installServiceLocked()`。

```
    // 加载插件，获取Service对象，并将其缓存起来
    private boolean installServiceLocked(ServiceRecord sr) {
        // 通过ServiceInfo创建Service对象
        Context plgc = Factory.queryPluginContext(sr.plugin);

        ClassLoader cl = plgc.getClassLoader();
        ....
        // 构建Service对象
        Service s;
        try {
            s = (Service) cl.loadClass(sr.serviceInfo.name).newInstance();
        } catch (Throwable e) {
            if (LOGR) {
                LogRelease.e(TAG, "isl: ni f " + sr.plugin, e);
            }
            return false;
        }

        // 只复写Context，别的都不做
        attachBaseContextLocked(s, plgc);

        s.onCreate();
        sr.service = s;

        // 开启“坑位”服务，防止进程被杀
        ComponentName pitCN = getPitComponentName();
        sr.pitComponentName = pitCN;
        startPitService(pitCN);
        return true;
    }
```

可以看到，使用插件的ClassLoader加载了Service，然后attach了Context，并且调用了service的`onCreate`。方法。

经过上面的分析我们可以简单的对Replugin中Service启动做这样一个总结：*Replugin通过复写`PluginContext`的`startService`, 根据要启动的Service的进程获取对应进程的`IPluginServiceServer`。在这个进程中使用`ServiceRecord`保存Service的信息，并模拟Service的运行逻辑。*


## BroadcastReceiver

对于BroadCastReceiver，其实在插件加载时就会处理 `Loader.loadDex()`

```
    // 创建插件的Component列表
    mComponents = Plugin.queryCachedComponentList(mPath);
    if (mComponents == null) {
        // ComponentList
        mComponents = new ComponentList(mPackageInfo, mPath, mPluginObj.mInfo);
        // 动态注册插件中声明的 receiver
        regReceivers();
        //...
    }
```

可以看出Replugin采用的方案也是把静态广播进行动态注册。

```
    private void regReceivers() throws android.os.RemoteException {
        String plugin = mPluginObj.mInfo.getName();

        if (mPluginHost == null) {
            mPluginHost = getPluginHost();
        }

        if (mPluginHost != null) {
            mPluginHost.regReceiver(plugin, ManifestParser.INS.getReceiverFilterMap(plugin));
        }
    }
```

这里的`getPluginHost`,获得的就是 `PROCESS_PERSIST`常驻进程的`PmHostSvc`。他是一个进程通信的`Binder`对象。

```
    @Override
    public void regReceiver(String plugin, Map rcvFilMap) throws RemoteException {
        HashMap<String, List<IntentFilter>> receiverFilterMap = (HashMap<String, List<IntentFilter>>) rcvFilMap;

        // 遍历此插件中所有静态声明的 Receiver
        for (HashMap.Entry<String, List<IntentFilter>> entry : receiverFilterMap.entrySet()) {
            if (mReceiverProxy == null) {
                mReceiverProxy = new PluginReceiverProxy();
                mReceiverProxy.setActionPluginMap(mActionPluginComponents);
            }

            /* 保存 action-plugin-receiver 的关系 */
            String receiver = entry.getKey();
            List<IntentFilter> filters = entry.getValue();

            if (filters != null) {
                for (IntentFilter filter : filters) {
                    .....
                    // 注册 Receiver
                    mContext.registerReceiver(mReceiverProxy, filter);
                }
            }
        }
    }
```

可以看到，所有的Receiver都被以`PluginReceiverProxy`代理注册。因此对于插件Receiver的注册实际上注册的是`PluginReceiverProxy`。在这个类中，收到广播时，会根据Receiver的进程，转到对应进程去处理：

```
public class PluginReceiverProxy extends BroadcastReceiver {
    //保存 Receiver 与 process 的关系
    private final HashMap<String, Integer> mReceiverProcess = new HashMap<>();

    @Override
    public void onReceive(Context context, Intent intent) {
        // 根据 action 取得 map<plugin, List<receiver>>
        HashMap<String, List<String>> pc = mActionPluginComponents.get(action);
        if (pc != null) {
            // 遍历每一个插件
            for (HashMap.Entry<String, List<String>> entry : pc.entrySet()) {
                String plugin = entry.getKey();
                if (entry.getValue() == null) {
                    continue;
                }

                // 拷贝数据，防止多线程问题
                List<String> receivers = new ArrayList<>(entry.getValue());
                // 此插件所有声明的 receiver
                for (String receiver : receivers) {

                        // 在对应进程接收广播, 如果进程未启动，则拉起之
                        int process = getProcessOfReceiver(plugin, receiver);

                        // todo 合并 IPluginClient 和 IPluginHost
                        if (process == IPluginManager.PROCESS_PERSIST) {
                            IPluginHost host = PluginProcessMain.getPluginHost();
                            host.onReceive(plugin, receiver, intent);
                        } else {
                            IPluginClient client = MP.startPluginProcess(plugin, process, new PluginBinderInfo(PluginBinderInfo.NONE_REQUEST));
                            client.onReceive(plugin, receiver, intent);
                        }
                }
            }
        }
    }
}
```

最终在目标进程调用真正`Receiver`的`onReceiver`方法

```
public class PluginReceiverHelper {

    public static void onPluginReceiverReceived(final String plugin,
                                                final String receiverName,
                                                final HashMap<String, BroadcastReceiver> receivers,
                                                final Intent intent) {
        .....
        // 使用插件的 Context 对象
        final Context pContext = Factory.queryPluginContext(plugin);
        if (pContext == null) {
            return;
        }

        String key = String.format("%s-%s", plugin, receiverName);

        BroadcastReceiver receiver = null;
        if (receivers == null || !receivers.containsKey(key)) {
            // 使用插件的 ClassLoader 加载 BroadcastReceiver
            Class c = loadClassSafety(pContext.getClassLoader(), receiverName);
            if (c != null) {
                receiver = (BroadcastReceiver) c.newInstance();
                if (receivers != null) {
                    receivers.put(key, receiver);
                }
            }
        } else {
            receiver = receivers.get(key);
        }

        if (receiver != null) {
            final BroadcastReceiver finalReceiver = receiver;
            Tasks.post2UI(new Runnable() {    // 转到 ui 线程 -> 如果不是主进程会运行在进程的Handler线程中
                @Override
                public void run() {
                    finalReceiver.onReceive(pContext, intent);
                }
            });
        }
    }
}
```
经过上面的分析，做一个简单的总结：*RePlugin对于所有的BroadcastReceiver都转化为动态Receiver，并使用`PluginReceiverProxy`来代理。当真正收到广播时，`PluginReceiverProxy`会实例化真正的Receiver，并调用`onReceive`方法。*

## ContentProvider

在RePlugin插件中无论是客户端还是服务端，都可以使用`PluginProviderClient`来进行ContentProvider的CRUD操作。这里以 query 方法为例来看一下:

```
    public static Cursor query(Context c, Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
        Uri turi = toCalledUri(c, uri);
        return c.getContentResolver().query(turi, projection, selection, selectionArgs, sortOrder);
    }

    /**
    * 将从【当前】插件或主程序里的URI转化成系统传过来的URI，且由插件Manifest来指定进程。例如：
    * Before:  content://com.qihoo360.contacts.abc/people （Contacts插件，UI）
    * After:   content://com.qihoo360.mobilesafe.PluginUIP/contacts/com.qihoo360.mobilesafe.contacts.abc/people
    *
    * @param c   当前的Context对象。若传递主程序的Context，则直接返回Uri，不作处理。否则就做Uri转换
    * @param uri URI对象
    * @return 转换后可直接在ContentResolver使用的URI
    */
    public static Uri toCalledUri(Context c, Uri uri) {
        String pn = fetchPluginByContext(c, uri);
        if (pn == null) {
            return uri;
        }
        return toCalledUri(c, pn, uri, IPluginManager.PROCESS_AUTO);
    }
```

上面注释已经解释的比较清楚了，如果是插件的PluginContext是可以获取到插件的信息的，会调用`toCalledUri()`。如果是主程序则走正常流程。下面主要看一下`toCalledUri`干了什么:

```
   public static Uri toCalledUri(Context context, String plugin, Uri uri, int process) {
        ....

        String au;
        if (process == IPluginManager.PROCESS_PERSIST) {
            au = PluginPitProviderPersist.AUTHORITY;
        } else if (PluginProcessHost.isCustomPluginProcess(process)) {
            au = PluginProcessHost.PROCESS_AUTHORITY_MAP.get(process);
        } else {
            au = PluginPitProviderUI.AUTHORITY;
        }

        // from => content://                                                  com.qihoo360.contacts.abc/people?id=9
        // to   => content://com.qihoo360.mobilesafe.Plugin.NP.UIP/plugin_name/com.qihoo360.contacts.abc/people?id=9
        String newUri = String.format("content://%s/%s/%s", au, plugin, uri.toString().replace("content://", ""));
        return Uri.parse(newUri);
    }

```
分析上面实现，如果你指定了ContentProvider的运行进程，则会得到不同的*authority*。

`String.format("content://%s/%s/%s", au, plugin, uri.toString().replace("content://", ""))` 这个就很明确了，即把原来的uri拼在最后面，重新组织uri的`authority`。这样替换后，再进行CRUD时，会把CRUD根据进程分配到下面5个Provider之一：

1. PluginPitProviderPersist （常驻进程的Provider）
2. PluginPitProviderUI      （主进程的Provider）
3. PluginPitProviderP1 、PluginPitProviderP2 、PluginPitProviderP2  （自定义进程的Provider）

其实上面这5个Provider都继承自 PluginPitProviderBase。 只不过 authority 不同。 因此继续看`PluginPitProviderBase`的query方法:

```
    @Override
    public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
        //转化为最原始的URI
        PluginProviderHelper.PluginUri pu = mHelper.toPluginUri(uri);
        //获取真正要执行的Provider
        ContentProvider cp = mHelper.getProvider(pu);
        //执行Query
        return cp.query(pu.transferredUri, projection, selection, selectionArgs, sortOrder);
    }
```

惯例，来个简单总结：*RePlugin中的ProviderCRUD时都会经由`PluginPitProviderBase`来做一个中转，`PluginPitProviderBase`会获取要真正执行的插件中的Provier，并调用相关CRUD方法。*




