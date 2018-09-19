---
title: VirtualApk-插件类加载与资源加载
date: 2018/6/05
tags: 
  - 插件化
  - susion
---

对于插件，在打包时并不会把其class文件和资源一起打入到主项目中，但如何在插件运行时获取到插件的类或资源呢？看一下VirtualApk的实现思路。插件资源和类的准备都是在插件构造时创建的。

```
   LoadedPlugin(PluginManager pluginManager, Context context, File apk) throws PackageParser.PackageParserException {
        this.mResources = createResources(context, apk); // 默认把资源添加到当前 ActivityThread 的 Resource Map中
        this.mClassLoader = createClassLoader(context, apk, this.mNativeLibDir, context.getClassLoader());
   }
```

## 插件的资源加载

```
    @WorkerThread
    private static Resources createResources(Context context, File apk) {
        if (Constants.COMBINE_RESOURCES) {
            //结合当前已加载的插件，构造出当前所有插件的资源
            Resources resources = ResourcesManager.createResources(context, apk.getAbsolutePath());
            //把资源添加到 activityThread Resources map中
            ResourcesManager.hookResources(context, resources);
            return resources;
        } else {
            Resources hostResources = context.getResources();
            AssetManager assetManager = createAssetManager(context, apk);
            return new Resources(assetManager, hostResources.getDisplayMetrics(), hostResources.getConfiguration());
        }
    }
```

可以看到这里把插件资源加载分成了两种模式：1.把插件的资源和宿主资源结合到一块，变成一个资源。 2.简单创建插件资源。

VirtualApk默认是将插件资源和宿主资源结合到一块的。

### 结合宿主资源 COMBINE_RESOURCES

```
 public static synchronized Resources createResources(Context hostContext, String apk) {
        Resources hostResources = hostContext.getResources();
        Resources newResources = null;
        AssetManager assetManager;
        try {

            //构建新的 AssetManager， 将宿主的资源和所有插件的资源添加到其中
            if (Build.VERSION.SDK_INT < Build.VERSION_CODES.LOLLIPOP) {
                assetManager = AssetManager.class.newInstance();
                ReflectUtil.invoke(AssetManager.class, assetManager, "addAssetPath", hostContext.getApplicationInfo().sourceDir);
            } else {
                assetManager = hostResources.getAssets();
            }

            //插件的 asset path  添加到 host中
            ReflectUtil.invoke(AssetManager.class, assetManager, "addAssetPath", apk);

            //每一次插件资源的加载，都要把以前加载过的资源整合一次
            List<LoadedPlugin> pluginList = PluginManager.getInstance(hostContext).getAllLoadedPlugins();
            for (LoadedPlugin plugin : pluginList) {
                ReflectUtil.invoke(AssetManager.class, assetManager, "addAssetPath", plugin.getLocation());
            }

            //对不同的平台做适配:不同的平台 Resource 类的名字并不相同
            if (isMiUi(hostResources)) {
                newResources = MiUiResourcesCompat.createResources(hostResources, assetManager);
            } else if (isVivo(hostResources)) {
                newResources = VivoResourcesCompat.createResources(hostContext, hostResources, assetManager);
            } else if (isNubia(hostResources)) {
                newResources = NubiaResourcesCompat.createResources(hostResources, assetManager);
            } else if (isNotRawResources(hostResources)) {
                newResources = AdaptationResourcesCompat.createResources(hostResources, assetManager);
            } else {
                // is raw android resources
                newResources = new Resources(assetManager, hostResources.getDisplayMetrics(), hostResources.getConfiguration());
            }

            //更新所有插件资源。 每一个插件可以访问：宿主 + 其它插件的资源 -> 其实插件是不可能访问其它插件的资源的，因为这个操作发生在运行时。
            for (LoadedPlugin plugin : pluginList) {
                //把插件资源设置给组件
                plugin.updateResources(newResources);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return newResources;
    }
```

对于VirtualApk上面这种做法，直接更新全部的 AssetManager和Resource，这是因为：

>由于有系统资源的存在，`mResources` 的初始化在很早就初始化了，所以我们就算通过`addAssetPath`方法将apk添加到`mAssetPaths`里，在查找资源的时候也不会找到这部分的资源，因为在旧的 `mResources` 里没有这部分的 id。所以在 Android L 之前是需要想办法构造一个新的`AssetManager`里的 `mResources`才行，这里有两种方案，VirtualAPK用的是类似 `InstantRun` 的那种方案，构造一个新的`AssetManager`，将宿主和加载过的插件的所有 apk 全都添加一遍，然后再调用`hookResources`方法将新的 `Resources` 替换回原来的，这样会引起两个问题，一个是每次加载新的插件都会重新构造一个 `AssetManger` 和 `Resources`，然后重新添加所有资源，这样涉及到很多机型的兼容(因为部分厂商自己修改了 `Resources` 的类名)，一个是需要有一个替换原来`Resources`的过程，这样就需要涉及到很多地方，看 `hookResources`方法。

>hookResources

```
    public static void hookResources(Context base, Resources resources) {
        try {
            //设置Application context的资源为新的资源
            ReflectUtil.setField(base.getClass(), base, "mResources", resources);
            //将主 apk 包中的资源更新为新的资源
            Object loadedApk = ReflectUtil.getPackageInfo(base);
            ReflectUtil.setField(loadedApk.getClass(), loadedApk, "mResources", resources);

            //替换 activityThread 的资源为新资源
            Object activityThread = ReflectUtil.getActivityThread(base);
            Object resManager = ReflectUtil.getField(activityThread.getClass(), activityThread, "mResourcesManager");

            if (Build.VERSION.SDK_INT < 24) {
                //private final ArrayMap<ResourcesKey, WeakReference<Resources> > mActiveResources = new ArrayMap<>();
                Map<Object, WeakReference<Resources>> map = (Map<Object, WeakReference<Resources>>) ReflectUtil.getField(resManager.getClass(), resManager, "mActiveResources");
                Object key = map.keySet().iterator().next();
                map.put(key, new WeakReference<>(resources));
            } else {
                // still hook Android N Resources, even though it's unnecessary, then nobody will be strange.
                Map map = (Map) ReflectUtil.getFieldNoException(resManager.getClass(), resManager, "mResourceImpls");
                Object key = map.keySet().iterator().next();
                Object resourcesImpl = ReflectUtil.getFieldNoException(Resources.class, resources, "mResourcesImpl");
                map.put(key, new WeakReference<>(resourcesImpl));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```
### 仅仅创建本插件资源

```
    AssetManager assetManager = createAssetManager(context, apk);
    return new Resources(assetManager, hostResources.getDisplayMetrics(), hostResources.getConfiguration());
```

这个操作还是很直接的，新建一个插件的 AssetManager, 然后利用它创建一份资源，然后返回。

### 插件使用已加载的资源

资源加载进来后，插件如何使用呢？

> PluginContext, 对于这个类我们已经知道它的作用了:插件中运行的四大组件的上下文基本都是它,看它的 `getResource` 方法。

```
    @Override
    public Resources getResources() {
        return this.mPlugin.getResources();
    }
```

## 插件的类加载

在`LoadedPlugin`创建时，会创建插件的 ClassLoader。

```
this.mClassLoader = createClassLoader(context, apk, this.mNativeLibDir, context.getClassLoader());
```

```
    private static ClassLoader createClassLoader(Context context, File apk, File libsDir, ClassLoader parent) {
        File dexOutputDir = context.getDir(Constants.OPTIMIZE_DIR, Context.MODE_PRIVATE);
        String dexOutputPath = dexOutputDir.getAbsolutePath();

        //DexClassLoader可以自定义 dex 加载的路径, 这里这个路径是: dexOutputPath。
        //parent为宿主的classloader。
        DexClassLoader loader = new DexClassLoader(apk.getAbsolutePath(), dexOutputPath, libsDir.getAbsolutePath(), parent);

        if (Constants.COMBINE_CLASSLOADER) {
            try {
                DexUtil.insertDex(loader);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        return loader;
    }
```

逻辑也不是很复杂，根据插件的apk创建插件的`DexClassLoader`,用来加载插件中的类。但是`Constants.COMBINE_CLASSLOADER`就有一点意思了，如果这个常量为true，则会将宿主的dex整合到一起。

>  DexUtil.insertDex(loader);

```
    public static void insertDex(DexClassLoader dexClassLoader) throws Exception {
        Object baseDexElements = getDexElements(getPathList(getPathClassLoader())); //获得宿主的dex列表
        Object newDexElements = getDexElements(getPathList(dexClassLoader)); //获得插件的dex列表

        //把插件的dex和宿主的dex整合到一起，并且宿主的dex位于列表的前面
        Object allDexElements = combineArray(baseDexElements, newDexElements);
        Object pathList = getPathList(getPathClassLoader());

        // 整合宿主和插件的classloader，然后重新设置 pathclassloader的 dexElements
        ReflectUtil.setField(pathList.getClass(), pathList, "dexElements", allDexElements);

        //设置新的native so 的path 到native加载列表中
        insertNativeLibrary(dexClassLoader);
    }
```

经过上面的操作，system， `PatchClassLoader`中就包含宿主的dex elements和插件的dex elements。并且每一个插件的classloader都可以加载本插件的类。

>insertNativeLibrary

对于本地so的操作和上面类似，也是整合宿主和插件的so:

```
    //把新的 so elements 设置到老的 so elements 后面
   for (int i = 0; i < newArrayLength; i++) {
                Object element = Array.get(newNativeLibraryPathElements, i);
                String dir = ((File)soPathField.get(element)).getAbsolutePath();
                if (dir.contains(Constants.NATIVE_DIR)) {
                    Array.set(allNativeLibraryPathElements, baseArrayLength, element);
                    break;
                }
            }

    //重新设置宿主的 native library
    ReflectUtil.setField(basePathList.getClass(), basePathList, "nativeLibraryPathElements", allNativeLibraryPathElements);
```





