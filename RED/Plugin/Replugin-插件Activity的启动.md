---
title: Replugin-插件Activity的启动
date: 2018/6/11
tags: 
  - 插件化
  - susion
---

## Hook 

>Replugin 相较于其他的插件化方案，只是hook了classload,下面看一下如何hook的。

`RePluginApplication.attachBaseContext() -> ..-> PMF.init() ->  PatchClassLoaderUtils.patch(application);`

```
    public static boolean patch(Application application) {

        // 获取Application的BaseContext （来自ContextWrapper）
        Context oBase = application.getBaseContext();
        //...
        Object oPackageInfo = ReflectUtils.readField(oBase, "mPackageInfo");
        //...
        // 获取mPackageInfo.mClassLoader
        ClassLoader oClassLoader = (ClassLoader) ReflectUtils.readField(oPackageInfo, "mClassLoader");
        //... 
        ClassLoader cl = RePlugin.getConfig().getCallbacks().createClassLoader(oClassLoader.getParent(), oClassLoader);

        // 将新的ClassLoader写入mPackageInfo.mClassLoader
        ReflectUtils.writeField(oPackageInfo, "mClassLoader", cl);

        // 设置线程上下文中的ClassLoader为RePluginClassLoader
        // 防止在个别Java库用到了Thread.currentThread().getContextClassLoader()时，“用了原来的PathClassLoader”，或为空指针
        Thread.currentThread().setContextClassLoader(cl);

        return true;
    }
```

可以看到上面就是把`Application`和UI线程的(PathcClassLoader)classloader替换为 `RePluginClassLoader` 。

### 重写 loadClass 方法

```
    @Override
    protected Class<?> loadClass(String className, boolean resolve) throws ClassNotFoundException {
        Class<?> c = null;
        c = PMF.loadClass(className, resolve);  //从坑中加载activity
        if (c != null) {
            return c;
        }
        try {
            c = mOrig.loadClass(className);
            // 只有开启“详细日志”才会输出，防止“刷屏”现象
            return c;
        } catch (Throwable e) {}

        return super.loadClass(className, resolve);
    }
```

可以看到这里优先使用 `PMF.loadClass(className, resolve);` 。在这个方法中是Replugin 占坑式加载Activity的真正实现。这里先不看。

## 插件Activity的启动

大致实现: 当启动一个插件中的Activity时，如果这个插件没有加载则加载。如果加载则最终会使用`IPluginClient.allocActivityContainer`, 为要加载的Activity分配一个对应的Activity坑位。把要启动的intent转化为这个坑位Activity的intent，并把目标要启动的Activity的信息保存在intent中，然后启动这个坑位的Activity。由于已经hook了Application的ClassLoader。所有在加载这个Activity类时。会把要加载的坑位Activity替换为目标表Acitvity。然后走正常的启动流程。


>由于代码太多，这里不一步一步分析。只看几个关键点：

PluginLibraryInternalProxy.startActivity:

```
        //找到的坑中的 Component
        ComponentName cn = mPluginMgr.mLocal.loadPluginActivity(intent, plugin, activity, process);
        if (cn == null) return false;

        // 将Intent指向到“坑位”。这样：
        // from：插件原Intent
        // to：坑位Intent
        intent.setComponent(cn);

        context.startActivity(intent);
```

>根据Activity要启动的进程，获取IPluginClent,进行坑位的分配

```
 public ComponentName loadPluginActivity(Intent intent, String plugin, String activity, int process) {

        ActivityInfo ai = null;
        String container = null;
        PluginBinderInfo info = new PluginBinderInfo(PluginBinderInfo.ACTIVITY_REQUEST);

        // 获取 ActivityInfo(可能是其它插件的 Activity，所以这里使用 pair 将 pluginName 也返回)
        ai = getActivityInfo(plugin, activity, intent);
        if (ai == null) return null;

        // 存储此 Activity 在插件 Manifest 中声明主题到 Intent
        intent.putExtra(INTENT_KEY_THEME_ID, ai.theme);

        // 根据 activity 的 processName，选择进程 ID 标识
        if (ai.processName != null) process = PluginClientHelper.getProcessInt(ai.processName);

        // 容器选择（启动目标进程）  process 为null时，在 UI进程启动
        IPluginClient client = MP.startPluginProcess(plugin, process, info);
        if (client == null) {
            return null;
        }

        // 远程分配坑位 container 为坑位activity的 class name
        container = client.allocActivityContainer(plugin, process, ai.name, intent);

        // 分配失败
        if (TextUtils.isEmpty(container))  return null;

        PmBase.cleanIntentPluginParams(intent);

        PluginIntent ii = new PluginIntent(intent);
        ii.setPlugin(plugin);
        ii.setActivity(ai.name);
        ii.setProcess(IPluginManager.PROCESS_AUTO);
        ii.setContainer(container);
        ii.setCounter(0);
        return new ComponentName(IPC.getPackageName(), container);
    }
```

`client.allocActivityContainer(plugin, process, ai.name, intent);`是一个进程间调用，具体实现类是`PluginProcessPer`。
在`allocActivityContainer()`方法中:

1. 如果没有加载插件则会进行插件的加载
2. 获取插件中此Activity的ActivityInfo
3. 根据Activity在manifest中声明的进程信息，在相应的PluginContainers中获取坑位

>下面以宿主进程来看一下如何获得Activity的坑位:

```
    final String alloc(ActivityInfo ai, String plugin, String activity, int process, Intent intent) {
        ActivityState state;

        String defaultPluginTaskAffinity = ai.applicationInfo.packageName;

        /* SingleInstance 优先级最高 */
        if (ai.launchMode == LAUNCH_SINGLE_INSTANCE) {
            synchronized (mLock) {
                state = allocLocked(ai, mLaunchModeStates.getStates(ai.launchMode, ai.theme), plugin, activity, intent);
            }

        /* TaskAffinity */
        } else if (!defaultPluginTaskAffinity.equals(ai.taskAffinity)) { // 非默认 taskAffinity
            synchronized (mLock) {
                state = allocLocked(ai, mTaskAffinityStates.getStates(ai), plugin, activity, intent);
            }

        /* SingleTask, SingleTop, Standard */
        } else {
            synchronized (mLock) {
                state = allocLocked(ai, mLaunchModeStates.getStates(ai.launchMode, ai.theme), plugin, activity, intent);
            }
        }

        if (state != null) {
            return state.container;
        }

        return null;
    }
```
可以看到，在Replugin中坑位的信息是保存在`LaunchModeStates`这种对象中的。这个对象维护了一个坑位的Map。对于每一个坑是这样表示的：

```
key : getInfix(launchMode, isTranslucentTheme(theme));
value :
        class ActivityState {
                final String container; //坑的class name
                int state; 
                String plugin;  //哪个插件的坑
                String activity; //坑对应的activity
                long timestamp;
        }
```

对于具体的分坑方法，可以了解一下:

```
 private final ActivityState allocLocked(ActivityInfo ai, HashMap<String, ActivityState> map,
                                            String plugin, String activity, Intent intent) {
        // 坑和状态的 map 为空
        if (map == null)return null;

        // 首先找上一个活的，或者已经注册的，避免多个坑到同一个activity的映射
        for (ActivityState state : map.values()) {
            if (state.isTarget(plugin, activity)) return state;
        }

        // 新分配：找空白的，第一个
        for (ActivityState state : map.values()) {
            if (state.state == STATE_NONE) {
                state.occupy(plugin, activity);
                return state;
            }
        }

        ActivityState found;

        // 重用：则找最老的那个
        found = null;
        for (ActivityState state : map.values()) {
            if (!state.hasRef()) {
                if (found == null) {
                    found = state;
                } else if (state.timestamp < found.timestamp) {
                    found = state;
                }
            }
        }
        if (found != null) {
            found.occupy(plugin, activity);
            return found;
        }

        // 强挤：最后一招，挤掉：最老的那个
        found = null;
        for (ActivityState state : map.values()) {
            if (found == null) {
                found = state;
            } else if (state.timestamp < found.timestamp) {
                found = state;
            }
        }

        if (found != null) {
            found.finishRefs();
            found.occupy(plugin, activity);
            return found;
        }

        // never reach here   66666
        return null;
    }
```

### Hook 点的作用

到这里已经找到要启动的activity与其对应的坑位。接下来就是hook点发挥作用了。按照前面的流程，在加载class时会使用 `PMF.loadClass(className, resolve);`这个方法最终会调用 `PmBase.loadClass()`

```
 final Class<?> loadClass(String className, boolean resolve) {
        //......

        if (mContainerActivities.contains(className)) {
            Class<?> c = mClient.resolveActivityClass(className);
            if (c != null) {
                return c;
            }
            return DummyActivity.class;
        }

        //...

        // 省略动态类的加载

        return loadDefaultClass(className);
    }
```

可以看出在加载时会去`mContainerActivities`容器中查找activity。那这个`mContainerActivities`是什么呢？其实在`PmBase`构造的时候会初始化这个容器：

```
   PmBase(Context context) {
        //...  
        mClient = new PluginProcessPer(context, this, PluginManager.sPluginProcessIndex, mContainerActivities);
        //... 
    }

    PluginProcessPer(Context context, PmBase pm, int process, HashSet<String> containers) {
        mContext = context;
        mPluginMgr = pm;
        mServiceMgr = new PluginServiceServer(context);

        mACM = new PluginContainers();
        mACM.init(process, containers);
    }
```

这下明白了`mContainerActivities`,实际上就是前面 `LaunchModeStates`的key的集合。因此如果要启动的activity和坑位做过映射，就在这个set中可以找到。找到之后，就会通过`PluginProcessPer`来加载真正的目标activity class。

```
    final Class<?> resolveActivityClass(String container) {
        String plugin = null;
        String activity = null;
        //...
        Plugin p = mPluginMgr.loadAppPlugin(plugin);
        //...
        ClassLoader cl = p.getClassLoader();
        //...
        Class<?> c = null;
        //...
        c = cl.loadClass(activity);
        //...
        return c;
    }
```

即在加载Activity时，会使用插件的Class loader来加载。


### 坑位Activity的来源

上面老是提到一些坑位Activity，那么这些坑位Activity，为什么可以经过AMS的校验，走到loadClass这一步的呢？其实Replugin的解决办法和许多插件化方案差不多，就是提前注册的manifest文件中，类似于下面这种形式：

```
        <activity
            android:name="${applicationId}.loader.a.Activity0"
            android:configChanges="keyboard|keyboardHidden|orientation|screenSize"
            android:exported="false"
            android:process=":loader0"
            android:screenOrientation="portrait"
            android:taskAffinity=":loader0"
            android:theme="@android:style/Theme.NoTitleBar" />

        <activity
            android:name="${applicationId}.loader.a.Activity0_fullscreen"
            android:configChanges="keyboard|keyboardHidden|orientation|screenSize"
            android:exported="false"
            android:process=":loader0"
            android:screenOrientation="portrait"
            android:theme="@android:style/Theme.NoTitleBar.Fullscreen" />
```

### Activity的 PluginContext

> 经过上面的分析，可能还是会有一个大大的疑问，那就是，插件activity在运行时资源、类启动怎么办？ 其实

所有的插件的Activity在编译时会被`replugin-plugin-gradle`转换为继承`PluginActivity`的子类:

```
public abstract class PluginActivity extends Activity {
    @Override
    protected void attachBaseContext(Context newBase) {
        newBase = RePluginInternal.createActivityContext(this, newBase);
        super.attachBaseContext(newBase);
    }
    ....
}
```
从上面代码可以看出，所有插件的Activity的Context都被替换为`PluginContext`。上面的方法最终会调用到宿主的：

```
    public Context createActivityContext(Activity activity, Context newBase) {
        // 此时插件必须被加载，因此通过class loader一定能找到对应的PLUGIN对象
        Plugin plugin = mPluginMgr.lookupPlugin(activity.getClass().getClassLoader());
        if (plugin == null) {
            if (LOG) {
                LogDebug.d(PLUGIN_TAG, "PACM: createActivityContext: can't found plugin object for activity=" + activity.getClass().getName());
            }
            return null;
        }
        return plugin.mLoader.createBaseContext(newBase);
    }

    final Context createBaseContext(Context newBase) {
        return new PluginContext(newBase, android.R.style.Theme, mClassLoader, mPkgResources, mPluginName, this);
    }

```
可以看到这个Context由插件的classloader，插件的资源构造而成。因此，插件的Activity运行就像正常Activity一样。

```
public class PluginContext extends ContextThemeWrapper {
    @Override
    public ClassLoader getClassLoader() {
        if (mNewClassLoader != null) {
            return mNewClassLoader;
        }
        return super.getClassLoader();
    }

    @Override
    public Resources getResources() {
        if (mNewResources != null) {
            return mNewResources;
        }
        return super.getResources();
    }

    @Override
    public AssetManager getAssets() {
        if (mNewResources != null) {
            return mNewResources.getAssets();
        }
        return super.getAssets();
    }
}

```

> ok 到这里大致分析了Replugin中对于插件activity加载的处理，细节问题还有很多，这里就不一一列出了。

