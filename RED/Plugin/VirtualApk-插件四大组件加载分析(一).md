---
title: VirtualApk-插件四大组件加载(一)
date: 2018/5/29
tags: 
  - 插件化
  - susion
---

> 这个系列的文章在网上其实有一些前辈写过了，但是我还是想自己分析一遍，主要是为了加深对实现细节的了解。

本文分析基于 VirtualApk 0.9.1。 在分析插件四大组件加载实现之前，先来了解一些VirtualAPK中的插件对象。

## LoadedPlugin与PluginManager

### LoadedPlugin

在VirtualApk中一个加载的插件是一个`LoadedPlugin`对象。 先来看一下`LoadedPlugin`的主要成员变量:

```
    //宿主host context
    private Context mHostContext;

    //自定义插件context
    private Context mPluginContext;

    //资源、类的管理
    private final File mNativeLibDir;
    private Resources mResources;
    private ClassLoader mClassLoader;
    
    //package info 管理
    private PluginPackageManager mPackageManager;
    private final PackageParser.Package mPackage;
    private final PackageInfo mPackageInfo;

    //四大组件，application，
    private Application mApplication;
    private Map<ComponentName, ActivityInfo> mActivityInfos;
    private Map<ComponentName, ServiceInfo> mServiceInfos;
    private Map<ComponentName, ActivityInfo> mReceiverInfos;
    private Map<ComponentName, ProviderInfo> mProviderInfos;
    private Map<String, ProviderInfo> mProviders; // key is authorities of provider
    private Map<ComponentName, InstrumentationInfo> mInstrumentationInfos;
```

从上面的这些变量可以看出，`LoadedPlugin`管理着自身的四大组件， 资源、类加载器等。

看一下`LoadedPlugin`的构造方法：
```
 LoadedPlugin(PluginManager pluginManager, Context context, File apk) throws Exception {
        //.....
        //解析 插件 apk 的package 信息
        this.mPackage = PackageParserCompat.parsePackage(context, apk, PackageParser.PARSE_MUST_BE_APK);
        //...
        //生成 PluginPackageManager 和 PluginContext
        this.mPackageManager = new PluginPackageManager();
        this.mPluginContext = new PluginContext(this);

        //构造插件资源和插件类加载器
        this.mNativeLibDir = context.getDir(Constants.NATIVE_DIR, Context.MODE_PRIVATE);
        this.mResources = createResources(context, apk);
        this.mClassLoader = createClassLoader(context, apk, this.mNativeLibDir, context.getClassLoader());
        tryToCopyNativeLib(apk);

        // Cache instrumentations
        Map<ComponentName, InstrumentationInfo> instrumentations = new HashMap<ComponentName, InstrumentationInfo>();
        for (PackageParser.Instrumentation instrumentation : this.mPackage.instrumentation) {
            instrumentations.put(instrumentation.getComponentName(), instrumentation.info);
        }
        this.mInstrumentationInfos = Collections.unmodifiableMap(instrumentations);
        this.mPackageInfo.instrumentation = instrumentations.values().toArray(new InstrumentationInfo[instrumentations.size()]);

        // 解析 apk 中的四大组件信息，并保存下来
        Map<ComponentName, ActivityInfo> activityInfos = new HashMap<ComponentName, ActivityInfo>();
        for (PackageParser.Activity activity : this.mPackage.activities) {
            activityInfos.put(activity.getComponentName(), activity.info);
        }
        ......
    }
```
在上面我们看到了`LoadedPlugin`中另外两个核心类：`PluginPackageManager` 与 `PluginContext`

#### PluginPackageManager

正如名称所示，这个类为插件的包管理器。对于`PackageManager`,我们可以获得一个APK的AndroidManifest中注册的信息、四大组件信息、权限等等配置。而`PluginPackageManager`复写了`PackageManager`的大多数方法，根据情况返回宿主的package info还是返回插件的package info。

比如对于插件的权限:
```
    @Override
    public PermissionInfo getPermissionInfo(String name, int flags) throws NameNotFoundException {
        //权限都要声明在宿主中吗？
        return this.mHostPackageManager.getPermissionInfo(name, flags);
    }
```
其实上面我也已经加了注释:权限都要声明在宿主中吗？

比如对于插件的Activity信息:
```
    @Override
    public ActivityInfo getActivityInfo(ComponentName component, int flags) throws NameNotFoundException {
        LoadedPlugin plugin = mPluginManager.getLoadedPlugin(component);
        if (null != plugin) {
            return plugin.mActivityInfos.get(component);
        }

        return this.mHostPackageManager.getActivityInfo(component, flags);
    }
```
可以看到，在调用获取activity信息时，`PluginPackageManager`并不是只查询当前插件中的activity信息，而是查询所有插件，如果没有，则查询宿主的PackageManager。

#### PluginContext

```
class PluginContext extends ContextWrapper
```

在插件的Actvity实际运行时，其Context就是这个对象，那么为什么要包一层呢，当然是希望对于Context中信息的获得，使用我们的实现,看几个 override的方法就一目了然了:

```
    @Override
    public ContentResolver getContentResolver() {
        return new PluginContentResolver(getHostContext());
    }

    @Override
    public Resources getResources() {
        return this.mPlugin.getResources();
    }

    @Override
    public AssetManager getAssets() {
        return this.mPlugin.getAssets();
    }

    @Override
    public Resources.Theme getTheme() {
        return this.mPlugin.getTheme();
    }

    @Override
    public void startActivity(Intent intent) {
        ComponentsHandler componentsHandler = mPlugin.getPluginManager().getComponentsHandler();
        componentsHandler.transformIntentToExplicitAsNeeded(intent);
        super.startActivity(intent);
    }
```

### PluginManager

`PluginManager`是VirtualApk中对已经加载了的插件进行管理的一个单例对象。它管理着VirtualAPK运行时的所有插件并负责插件四大组件的一部分的hook工作。

```
    private Map<String, LoadedPlugin> mPlugins = new ConcurrentHashMap<>();

    private Instrumentation mInstrumentation; // Hooked instrumentation
    private IActivityManager mActivityManager; // Hooked IActivityManager binder
    private IContentProvider mIContentProvider; // Hooked IContentProvider binder
```

为什么说负责四大组件的一部分的hack工作呢？

>由于四大组件都是需要在AndroidManifest中注册的，而插件apk中的组件是不可能预先知晓名字，提前注册中宿主apk中的，所以现在基本都采用一些hack方案类解决。

先来了解一下大体是如何hack的:

- Activity：在宿主apk中提前占几个坑，然后通过“欺上瞒下”（这个词好像是360之前的ppt中提到）的方式，启动插件apk的Activity；因为要支持不同的launchMode以及一些特殊的属性，需要占多个坑。
- Service：通过代理Service的方式去分发；主进程和其他进程，VirtualAPK使用了两个代理Service。
- BroadcastReceiver：静态转动态
- ContentProvider：通过一个代理Provider进行分发。

在构建`PluginManager`对象时就会hack系统的`Instrumentation`和`AMS`。

```
   hookInstrumentationAndHandler();
    if (Build.VERSION.SDK_INT >= 26) {
        hookAMSForO();
    } else {
        //hook ActivityManagerNative 的ActivityManager实例为自己的ActivityMananger
        hookSystemServices();
    }
```

具体为什么这么做下面具体分析时再说。

上面对VirtualApk中一个加载了的插件有一定的小了解，接下来看一下VirtualApk是如何支持插件的四大组件运作的:

## 插件中Activity的启动过程

大体实现方案: 在宿主apk中提前占几个坑，然后通过“欺上瞒下”的方式，启动插件apk的Activity。

对于"欺上瞒下"，实现的前提是hack了`Instrumentation`上面我们已经知道，在`PluginManager`构造时就会hack为`VAInstrumentation`。

```
public class VAInstrumentation extends Instrumentation implements Handler.Callback
```

```
    private void hookInstrumentationAndHandler() {
        try {
            Instrumentation baseInstrumentation = ReflectUtil.getInstrumentation(this.mContext);
            if (baseInstrumentation.getClass().getName().contains("lbe")) {
                // reject executing in paralell space, for example, lbe.
                System.exit(0);
            }

            final VAInstrumentation instrumentation = new VAInstrumentation(this, baseInstrumentation);
            Object activityThread = ReflectUtil.getActivityThread(this.mContext);
            ReflectUtil.setInstrumentation(activityThread, instrumentation);

            //Main handler 的callback 在 VAInstrumentation 中有回调
            ReflectUtil.setHandlerCallback(this.mContext, instrumentation);
            this.mInstrumentation = instrumentation;
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

即把系统的 `ActivityThread`的`Instrumentation`替换为`VAInstrumentation`。并把和`Application context`的HandlerCallbak`VAInstrumentation`。

在启动一个插件中的Activity会依次调用 `VAInstrumentation` 下面方法:

execStartActivity -> realExecStartActivity -> handleMessage LAUNCH_ACTIVITY -> newActivity -> callActivityOnCreate

下面来依次分析:

### execStartActivity

execStartActivity主要是AMS来校验要启动的Activity是否存在。`VAInstrumentation` hack 这个方法是为了在启动插件中的Activity时利用占位Activity来绕过Activity的校验问题。因为插件中的Activity并不能在宿主中做预注册。

```
    public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
        mPluginManager.getComponentsHandler().transformIntentToExplicitAsNeeded(intent);
        // null component is an implicitly intent
        if (intent.getComponent() != null) { //如果指定的启动activity为 host 或者 plugin 则 component不应该为null
            Log.i(TAG, String.format("execStartActivity[%s : %s]", intent.getComponent().getPackageName(),
                    intent.getComponent().getClassName()));
            // resolve intent with Stub Activity if needed
            //如果是组件的activity， 则利用预先缓存的activity来作为开启容器： 设置intent的class name 为预缓存的stub activity的name
            this.mPluginManager.getComponentsHandler().markIntentIfNeeded(intent);
        }

        ActivityResult result = realExecStartActivity(who, contextThread, token, target,
                    intent, requestCode, options);

        return result;

    }
```

>transformIntentToExplicitAsNeeded
这个方法的作用是:如果Activity是隐式启动，并且如果是组件中的Activity，则把这个Activity转为显示启动:

```
    ResolveInfo info = mPluginManager.resolveActivity(intent);
    if (info != null && info.activityInfo != null) {
        //info.activityInfo.name ->  class name
        component = new ComponentName(info.activityInfo.packageName, info.activityInfo.name);
        intent.setComponent(component);
    }
```

>markIntentIfNeeded
这个方法的作用是:如果是组件的Intent，则在intent中设置上一些组件的信息:

```
    // 要启动的intent位于插件中
    if (!targetPackageName.equals(mContext.getPackageName()) && mPluginManager.getLoadedPlugin(targetPackageName) != null) {
        intent.putExtra(Constants.KEY_IS_PLUGIN, true);
        intent.putExtra(Constants.KEY_TARGET_PACKAGE, targetPackageName);
        intent.putExtra(Constants.KEY_TARGET_ACTIVITY, targetClassName);
        dispatchStubActivity(intent);
    }
```

>dispatchStubActivity(intent)
这个方法的作用是:从宿主中获得启动模式和要启动的Activity的相同的占位Activity，并把intent要启动的Activity设置为这个占位Activity。

```
    private void dispatchStubActivity(Intent intent) {
        ComponentName component = intent.getComponent();
        String targetClassName = intent.getComponent().getClassName();
        LoadedPlugin loadedPlugin = mPluginManager.getLoadedPlugin(intent);
        ActivityInfo info = loadedPlugin.getActivityInfo(component);

        int launchMode = info.launchMode;
        Resources.Theme themeObj = loadedPlugin.getResources().newTheme();
        themeObj.applyStyle(info.theme, true);
        String stubActivity = mStubActivityInfo.getStubActivity(targetClassName, launchMode, themeObj);
        intent.setClassName(mContext, stubActivity);
    }
```
那么为什么要启动占位Activity呢？我们知道Activity是必须在manifest文件中注册的，插件中的Activity是不能预注册的，VirtualApk在manifest中预注册了一些Activity，来作为插件Activity启动的占位Activity:

```
    <!-- Stub Activities -->
    <activity android:name=".A$1" android:launchMode="standard"/>
    <activity android:name=".A$2" android:launchMode="standard"
        android:theme="@android:style/Theme.Translucent" />
```


###  realExecStartActivity

上面在包装好占位Activtiy后就会调用这个方法，这个方法会通过反射真正去调用被 hack 的 Instrumentation 的 `execStartActivity`方法。

```
    private ActivityResult realExecStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
        ActivityResult result = null;
        try {
            Class[] parameterTypes = {Context.class, IBinder.class, IBinder.class, Activity.class, Intent.class,
            int.class, Bundle.class};
            result = (ActivityResult)ReflectUtil.invoke(Instrumentation.class, mBase,"execStartActivity", parameterTypes,who, contextThread, token, target, intent, requestCode, options);
        } catch (Exception e) {
            if (e.getCause() instanceof ActivityNotFoundException) {
                throw (ActivityNotFoundException) e.getCause();
            }
            e.printStackTrace();
        }
        return result;
    }
```

###  Handler.Callback -> handleMessage

上面在绕过了对系统对Activity的校验后，会被Server通过Handler调用这个handleMessage方法:

```
    if (msg.what == LAUNCH_ACTIVITY) {
        Object r = msg.obj; // ActivityClientRecord r
        try {
            Intent intent = (Intent) ReflectUtil.getField(r.getClass(), r, "intent");
            //用来加载extra 中 Parcelable class
            intent.setExtrasClassLoader(VAInstrumentation.class.getClassLoader());
            ActivityInfo activityInfo = (ActivityInfo) ReflectUtil.getField(r.getClass(), r, "activityInfo");

            if (PluginUtil.isIntentFromPlugin(intent)) { // 在占位Activity的intent中会设置这个标记位
                int theme = PluginUtil.getTheme(mPluginManager.getHostContext(), intent);
                if (theme != 0) {
                    activityInfo.theme = theme;
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

可以看出这个方法主要是给插件的Activity设置theme。

### newActivity

```
    public Activity newActivity(ClassLoader cl, String className, Intent intent) throws InstantiationException, IllegalAccessException, ClassNotFoundException {
        //只要启动的是占坑的Activity， 必定会进入到异常中，去使用插件的classloader，来加载插件的类
        try {
            cl.loadClass(className);
        } catch (ClassNotFoundException e) {
            LoadedPlugin plugin = this.mPluginManager.getLoadedPlugin(intent);
            String targetClassName = PluginUtil.getTargetActivity(intent);

            if (targetClassName != null) {
                //使用组件的classloader来构造这个activity
                Activity activity = mBase.newActivity(plugin.getClassLoader(), targetClassName, intent);
                activity.setIntent(intent);

                try {
                    // for 4.1+
                    ReflectUtil.setField(ContextThemeWrapper.class, activity, "mResources", plugin.getResources());
                } catch (Exception ignored) {
                    // ignored.
                }
                return activity;
            }
        }
        return mBase.newActivity(cl, className, intent);
    }
```

这里使用主PathClassLoader来加载要启动的占位Activity, But 为什么主ClassLoader会找不到这个占位Activity呢??? (VirtualApk其实已经把插件的dex文件插入到主classloader中了) 暂时我还不知道。。。。

先放一下，这里使用插件的 DexClassLoader来加载插件的Activity，并设置好intent，启动插件Activity。

### callActivityOnCreate

```
    @Override
    public void callActivityOnCreate(Activity activity, Bundle icicle) {
        Log.e("wpc", "callActivityOnCreate");
        final Intent intent = activity.getIntent();
        if (PluginUtil.isIntentFromPlugin(intent)) {
            Context base = activity.getBaseContext();
            try {
                LoadedPlugin plugin = this.mPluginManager.getLoadedPlugin(intent);
                // 资源已经加载进来了，需要设置给activity
                ReflectUtil.setField(base.getClass(), base, "mResources", plugin.getResources());

                //给Activity设置对应的组件的 context 与 applicaiton
                ReflectUtil.setField(ContextWrapper.class, activity, "mBase", plugin.getPluginContext());
                ReflectUtil.setField(Activity.class, activity, "mApplication", plugin.getApplication());
                ReflectUtil.setFieldNoException(ContextThemeWrapper.class, activity, "mBase", plugin.getPluginContext());

                // set screenOrientation
                ActivityInfo activityInfo = plugin.getActivityInfo(PluginUtil.getComponent(intent));
                if (activityInfo.screenOrientation != ActivityInfo.SCREEN_ORIENTATION_UNSPECIFIED) {
                    activity.setRequestedOrientation(activityInfo.screenOrientation);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }

        }

        mBase.callActivityOnCreate(activity, icicle);
    }

```

在这个方法中替换了插件Activity的Context为 `PluginContex`, Application为插件的`Application`, （对，插件是会实例化自己的Application）并设置类Activity的 resources。 动态设置Activity的屏幕方向。
然后反射调用Activity的`onCreate`方法。





