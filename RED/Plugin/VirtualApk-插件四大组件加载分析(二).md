---
title: VirtualApk-插件四大组件加载(二)
date: 2018/6/04
tags: 
  - 插件化
  - susion
---

前一节分析了插件Activity的启动。接下来看一下 Service、BroadcastReceiver、ContentProvider。

## 插件Service的启动

实现大概原理:将系统的ActivityManager hack为`ActivityManagerProxy`。拦截所有Service的启动方法，启动`LocalService`或者`RemoteService`。在`LocalService`或`RemoteService`对Service的启动Action作统一的处理分发。

这两个Service已经在manifest中声明:

```
    <!-- Local Service running in main process -->
    <service android:name="com.didi.virtualapk.delegate.LocalService" />

    <!-- Daemon Service running in child process -->
    <service android:name="com.didi.virtualapk.delegate.RemoteService" android:process=":daemon">
        <intent-filter>
            <action android:name="${applicationId}.intent.ACTION_DAEMON_SERVICE" />
        </intent-filter>
    </service>
```

### hack系统的ActivityManager

>在`PluginManager`实例化时会做这个操作:

```
  if (Build.VERSION.SDK_INT >= 26) {
      this.hookAMSForO();
  } else {
      this.hookSystemServices();
  }

  private void hookSystemServices() {
      try {
          Singleton<IActivityManager> defaultSingleton = (Singleton<IActivityManager>) ReflectUtil.getField(ActivityManagerNative.class, null, "gDefault");
          IActivityManager activityManagerProxy = ActivityManagerProxy.newInstance(this, defaultSingleton.get());

          ReflectUtil.setField(defaultSingleton.getClass().getSuperclass(), defaultSingleton, "mInstance", activityManagerProxy);

          if (defaultSingleton.get() == activityManagerProxy) {
              this.mActivityManager = activityManagerProxy;
          }
      } catch (Exception e) {
          e.printStackTrace();
      }
  }

  public class ActivityManagerProxy implements InvocationHandler
  ```
### ActivityManagerProxy对于启动Service的拦截

```
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if ("startService".equals(method.getName())) {
          ...
           return startService(proxy, method, args);
        }
        ...
        } else if ("bindService".equals(method.getName())) {
          ...
          return bindService(proxy, method, args);
        } 
    
        //不是插件的service操作，则直接交由ActivityManager处理
        return method.invoke(this.mActivityManager, args);
    }
```

> startService()

这个方法内部处理：如果启动的service不是插件service，则走正常的启动流程，否则调用`startDelegateServiceForTarget`

```
    private ComponentName startDelegateServiceForTarget(Intent target, ServiceInfo serviceInfo, Bundle extras, int command) {
        Intent wrapperIntent = wrapperTargetIntent(target, serviceInfo, extras, command);
        return mPluginManager.getHostContext().startService(wrapperIntent);
    }

    private Intent wrapperTargetIntent(Intent target, ServiceInfo serviceInfo, Bundle extras, int command) {
        // fill in service with ComponentName
        target.setComponent(new ComponentName(serviceInfo.packageName, serviceInfo.name));
        String pluginLocation = mPluginManager.getLoadedPlugin(target.getComponent()).getLocation();

        // start delegate service to run plugin service inside
        boolean local = PluginUtil.isLocalService(serviceInfo); //通过包名或插件的包名来判断
        Class<? extends Service> delegate = local ? LocalService.class : RemoteService.class;
        Intent intent = new Intent();
        intent.setClass(mPluginManager.getHostContext(), delegate);
        intent.putExtra(RemoteService.EXTRA_TARGET, target);
        intent.putExtra(RemoteService.EXTRA_COMMAND, command);
        intent.putExtra(RemoteService.EXTRA_PLUGIN_LOCATION, pluginLocation);
        if (extras != null) {
            intent.putExtras(extras);
        }
        return intent;
    }
```
可以看到把要启动的service的信息包装在`LocalService`,启动`LocalService`。

即对于要启动插件Service的所有方法：`startService` `stopService`  `bindService` `unbindService`都是通过启动`LocalService`来实现。

### LocalService

这个Service可以当做一个 插件Service操作的处理者，比如对于Start一个插件Service，在这个类中会实例化插件Service，并模拟调用其相关生命周期方法:

```
   switch (command) {
            case EXTRA_COMMAND_START_SERVICE: {
                ActivityThread mainThread = (ActivityThread)ReflectUtil.getActivityThread(getBaseContext());
                IApplicationThread appThread = mainThread.getApplicationThread();
                Service service;

                //如果Service已经启动过了，则直接获取。
                if (this.mPluginManager.getComponentsHandler().isServiceAvailable(component)) {
                    service = this.mPluginManager.getComponentsHandler().getService(component);
                } else {
                    try {
                        //Service第一次启动，要调用其生命周期相关方法。
                        service = (Service) plugin.getClassLoader().loadClass(component.getClassName()).newInstance();

                        Application app = plugin.getApplication();
                        IBinder token = appThread.asBinder();
                        Method attach = service.getClass().getMethod("attach", Context.class, ActivityThread.class, String.class, IBinder.class, Application.class, Object.class);
                        IActivityManager am = mPluginManager.getActivityManager();
                        //service.attch()是一个 @hide方法，需要反射调用
                        attach.invoke(service, plugin.getPluginContext(), mainThread, component.getClassName(), token, app, am);
                        //onCreate
                        service.onCreate();
                        this.mPluginManager.getComponentsHandler().rememberService(component, service);
                    } catch (Throwable t) {
                        return START_STICKY;
                    }
                }
                //对应 -> EXTRA_COMMAND_START_SERVICE
                service.onStartCommand(target, 0, this.mPluginManager.getComponentsHandler().getServiceCounter(service).getAndIncrement());
                break;
            }
```
对于其他方法类似于 `EXTRA_COMMAND_START_SERVICE`。

*可以看到对于插件Service的核心实现是，利用宿主的`LocalService`或`RemoteService`来运行插件Service的方法*

## 插件BroadcastReceiver的处理

对于BroadcastReceiver的处理方式还是十分简单的：在加载插件时，把所有静态注册的Receiver转化为动态Receiver并注册

```
    // Register broadcast receivers dynamically
    //所有broadcast receiver 动态注册到宿主中
    Map<ComponentName, ActivityInfo> receivers = new HashMap<ComponentName, ActivityInfo>();
    for (PackageParser.Activity receiver : this.mPackage.receivers) {
        receivers.put(receiver.getComponentName(), receiver.info);

        try {
            BroadcastReceiver br = BroadcastReceiver.class.cast(getClassLoader().loadClass(receiver.getComponentName().getClassName()).newInstance());
            for (PackageParser.ActivityIntentInfo aii : receiver.intents) {
                this.mHostContext.registerReceiver(br, aii);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    this.mReceiverInfos = Collections.unmodifiableMap(receivers);
    this.mPackageInfo.receivers = receivers.values().toArray(new ActivityInfo[receivers.size()]);

```

## 插件ContentProvider的处理

这个实现还是比较有意思的:使用`IContentProviderProxy` hack 某一Uri规则的ContentProvider，对所有启动的ContentProvider的Uri都做一层处理，使最终处理的ContentProvider为:`RemoteContentProvider`。
再由``RemoteContentProvider`来做分发。

### hack的起点

前面我们已经知道，所有的插件组件的Context为`PluginContext`。

```
class PluginContext extends ContextWrapper {

    .....
    @Override
    public ContentResolver getContentResolver() {
        return new PluginContentResolver(getHostContext());
    }

}
```

系统调用`PluginContentResolver`的任意一个方法，都会对ContentProvider做hack。

```
 protected IContentProvider acquireProvider(Context context, String auth) {
        try {
            //如果是插件的content provider 则返回hook的 IContentProvider
            if (mPluginManager.resolveContentProvider(auth, 0) != null) {
                return mPluginManager.getIContentProvider();
            }

            //直接调用宿主的
            return (IContentProvider) sAcquireProvider.invoke(mBase, context, auth);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

   public synchronized IContentProvider getIContentProvider() {
        if (mIContentProvider == null) {
            hookIContentProviderAsNeeded();
        }
        return mIContentProvider;
    }
```
那么是如何hack的呢？hack的是那些ContentProvider呢？

```
  private void hookIContentProviderAsNeeded() {
        ....
        //context.getPackageName() + ".VirtualAPK.Provider";
        if (auth.equals(PluginContentResolver.getAuthority(mContext))) {
            if (mProvider == null) {
                mProvider = val.getClass().getDeclaredField("mProvider");
                mProvider.setAccessible(true);
            }
            IContentProvider rawProvider = (IContentProvider) mProvider.get(val);
            //直接 hook 为 ->IContentProviderProxy
            IContentProvider proxy = IContentProviderProxy.newInstance(mContext, rawProvider);
            mIContentProvider = proxy;
        ...
    }
```
即这里hack的为 uri 为 `context.getPackageName() + ".VirtualAPK.Provider"`的ContentProvider。hack 为`IContentProviderProxy`

### IContentProviderProxy的转到RemoteContentProvider

```
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //主项目在调用content provider相关方法时， 会先给 uri wrapper一层， 插件信息 -> wrapper 的目的是，最终会启动 RemoteContentProvider。
        wrapperUri(method, args);
        return method.invoke(mBase, args);
    }
```

>wrapperUri

这个方法把ContentProvider的uri重新组织，指向`RemoteContentProvider`, 并把原来的uri拼接在最后面

```
  //对于每个URI添加上插件参数
    PluginManager pluginManager = PluginManager.getInstance(mContext);
    ProviderInfo info = pluginManager.resolveContentProvider(uri.getAuthority(), 0);
    if (info != null) {
        String pkg = info.packageName;
        LoadedPlugin plugin = pluginManager.getLoadedPlugin(pkg);
        String pluginUri = Uri.encode(uri.toString());
        //base uri 是 : RemoteContentProvider
        StringBuilder builder = new StringBuilder(PluginContentResolver.getUri(mContext));
        builder.append("/?plugin=" + plugin.getLocation());
        builder.append("&pkg=" + pkg);
        builder.append("&uri=" + pluginUri);
        Uri wrapperUri = Uri.parse(builder.toString());
        if (method.getName().equals("call")) {
            bundleInCallMethod.putString(KEY_WRAPPER_URI, wrapperUri.toString());
        } else {
            args[index] = wrapperUri;
        }
    }
```
为什么说指向`RemoteContentProvider`呢？

```
    @Deprecated
    public static String getUri(Context context) {
        return "content://" + getAuthority(context);
    }

    <provider
    android:name="com.didi.virtualapk.delegate.RemoteContentProvider"
    android:authorities="${applicationId}.VirtualAPK.Provider"
    android:process=":daemon" />
```

### RemoteContentProvider

它的行为就很直接了：实例化真正的ContentProvider，并做相应的操作:

```

    @Override
    public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
        ContentProvider provider = getContentProvider(uri);
        Uri pluginUri = Uri.parse(uri.getQueryParameter(KEY_URI));
        if (provider != null) {
            return provider.query(pluginUri, projection, selection, selectionArgs, sortOrder);
        }

        return null;
    }

    //在这个方法中，分发每个组件的 ContentProvider
    private ContentProvider getContentProvider(final Uri uri) {
        final PluginManager pluginManager = PluginManager.getInstance(getContext());
        Uri pluginUri = Uri.parse(uri.getQueryParameter(KEY_URI));
        final String auth = pluginUri.getAuthority();
        ContentProvider cachedProvider = sCachedProviders.get(auth);
        if (cachedProvider != null) {
            return cachedProvider;
        }

        synchronized (sCachedProviders) {
            LoadedPlugin plugin = pluginManager.getLoadedPlugin(uri.getQueryParameter(KEY_PKG));
            if (plugin == null) {
                try {
                    pluginManager.loadPlugin(new File(uri.getQueryParameter(KEY_PLUGIN)));
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }

            final ProviderInfo providerInfo = pluginManager.resolveContentProvider(auth, 0);
            if (providerInfo != null) {
                RunUtil.runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            LoadedPlugin loadedPlugin = pluginManager.getLoadedPlugin(uri.getQueryParameter(KEY_PKG));
                            ContentProvider contentProvider = (ContentProvider) Class.forName(providerInfo.name).newInstance();
                            contentProvider.attachInfo(loadedPlugin.getPluginContext(), providerInfo);
                            sCachedProviders.put(auth, contentProvider);
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                }, true);
                return sCachedProviders.get(auth);
            }
        }

        return null;
    }

```