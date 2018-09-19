---
title: Android Architecture Components 之 Lifecycle、LiveData、ViewModel
date: 2018/3/27
tags: 
  - ArchitectureComponents 
  - susion
---


>本文仅是阅读[Android Architecture Components](https://developer.android.google.cn/topic/libraries/architecture/guide.html)的个人总结。

- Android App开发的痛点

在开发AndroidApp时，由于系统的UI Controller（Activity、Fragment）Component能够被独立启动并且是无顺序的，他们的生命周期不受开发者的控制，因此当我们的数据依赖这些组件的生命周期时就会发生许多问题。

Android官方指出一个重要的开发原则是:`我们不应该把不是处理UI和系统事件的代码写在UI Controller中。`

>Any code that does not handle a UI or operating system interaction should not be in these classes. Keeping them as lean as possible will allow you to avoid many lifecycle related problems. Don't forget that you don't own those classes, they are just glue classes that embody the contract between the OS and your app.

官方指出第二个重要的原则是`drive your UI from a model`

`Architecture Components`就是遵守上面两个原则的框架，它保证我们app可以很好的响应系统组件的生命周期，不出现混乱的情况。同时`drive your UI from a model`

### Handling Lifecycles with Lifecycle-Aware Components

这个其实原理是很简单的：UI Controller(Activity、Fragment)是一个`LifecycleOwner`,它的生命周期时可以被观察的。我们可以向它们上面注册`LifecycleObserver`，观察生命周期事件。简单代码如下：


创建一个`LifecycleObserver`:

```
public class MainPresenter implements LifecycleObserver {

    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    void initView() {
        Log.e(TAG, "init View");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    void initData() {
        Log.e(TAG, "initData");
    }

}
```

向`LifecycleOwner`注册`LifecycleObserver`:

```
public class MainActivity extends AppCompatActivity implements MainView {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState)
        getLifecycle().addObserver(new MainPresenter());
    }

```

> note: 在` Support Library 26.1.0`, `AppCompatActivity`和`Fragment` 都已经实现`LifecycleOwner`

对于上面这个sample，可以说简单的把UI Controller的生命周期相关方法dispatch给其他组件处理，使代码看起来非常清晰。

### LiveData

先看一下使用`LiveData`编码的优势:

- 确保UI状态和数据状态是同步的。
- 没有内存泄漏
- 不会由于Activity的Stop而引起crash
- 不需要手动处理组件生命周期
- 不会由于系统config改变而保持数据，比如屏幕旋转


`LiveData`是一个包裹数据并且可以被观察的对象，它可以感知App组件的生命周期（`Activity Fragment Service`）。这个特性使:`LiveData可以在组件活跃的状态(在生命周期内)下更新数据`

>活跃状态是指`START`or`RESUMED`。即LiveData在数据发生改变时只把这个改变事件通知给活跃的观察者，对于不活跃的观察者是不会通知的。

在给`LiveData`注册`Observer`时，可以把`LifecycleOwner`和`LiveData`组成一个配对关系。这样在`LifecycleOwner`不活跃时，`Observer`会被移除。这样就保证像`Activity Fragment`在观察`LiveData`变化时是非常安全的，即在不活跃时，是不会响应`LiveData`的变化的。

下面看一下如何使用：

```
public class NameViewModel extends ViewModel {

    // Create a LiveData with a String
    private MutableLiveData<String> mCurrentName;
    
    public MutableLiveData<String> getCurrentName() {
        if (mCurrentName == null) {
            mCurrentName = new MutableLiveData<String>();
        }
        return mCurrentName;
    }
}

```
`MutableLiveData`是一个`LiveData`,是一个可被观察的对象。官方是推荐把`LiveData`写在`ViewModel`中的：
>- 可以避免Activity和Fragment的臃肿，这些UI Controller只是响应UI的变化，而不会持有UI的状态
>- 解耦LiveData和UIController
对于`ViewModel`将会在下面介绍。


通过`observer()`方法将`LifecycleOwner和LiveData`建立联系:

```

public class NameActivity extends AppCompatActivity {

    private NameViewModel mModel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Get the ViewModel.
        mModel = ViewModelProviders.of(this).get(NameViewModel.class);

        // Create the observer which updates the UI.
        final Observer<String> nameObserver = new Observer<String>() {
            @Override
            public void onChanged(@Nullable final String newName) {
                // Update the UI, in this case, a TextView.
                mNameTextView.setText(newName);
            }
        };
});

        // Observe the LiveData, passing in this activity as the LifecycleOwner and the observer.
        mModel.getCurrentName().observe(this, nameObserver);
    }
}
```

上面把一个`Observer`加入到`LiveData`的观察者列表，如果`Oberver`对应的`LifeOwner`(UI Controller)不活跃了，这个`Observer`将会被移除。

数据变化，触发活跃的`LifeOwner`更新UI,即调用`Observer`的`onChanged()`方法。

```
mButton.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
        String anotherName = "John Doe";
        mModel.getCurrentName().setValue(anotherName);
    }
});
```


### ViewModel

这个类被设计用来管理和UI相关的数据(即`LiveData`),它允许数据感知系统的配置变化，比如屏幕旋转。

>ViewModel在发生屏幕旋转事件是，这个对象会被保留，如果此时UI Controller销毁了(Activity)，在Activity实例重建的时候，这个对象是可以被重新获得的，即我们可以不用在` onSaveInstanceState() `保存数据，然后在`onCreate()`中恢复。

可以这样获得一个`ViewModel`

```
public class MyViewModel extends ViewModel {
    private MutableLiveData<List<User>> users;
    public LiveData<List<User>> getUsers() {
        if (users == null) {
            users = new MutableLiveData<List<Users>>();
            loadUsers();
        }
        return users;
    }

    private void loadUsers() {
        // Do an asynchronous operation to fetch users.
    }
    
    @Override
    protected void onCleared() {
        super.onCleared();
    }
}

public class MyActivity extends AppCompatActivity {
    public void onCreate(Bundle savedInstanceState) {
        // Create a ViewModel the first time the system calls an activity's onCreate() method.
        // Re-created activities receive the same MyViewModel instance created by the first activity.

        MyViewModel model = ViewModelProviders.of(this).get(MyViewModel.class);
        model.getUsers().observe(this, users -> {
            // update UI
        });
    }
}
```



ViewModel的生命周期如下：

![](http://7xrbxa.com1.z0.glb.clouddn.com/galaxywarshipviewmodel-lifecycle.png)

即这个对象作用域直到Activity destroy。当框架层面Activity被销毁时，`ViewModel`的`onCleared`方法会被调用。

官方建议：`ViewModel`不应该引用任何UI Controller相关对象。它也不应该关心任何UI Controller的生命周期变化。

如果你需要一个 `Application Context`，可以使用 ` AndroidViewModel `


#### 在多个Fragment中的应用
在上面可以看出一个`ViewModel`其实适合`LifecycleOwner`绑定的，因此Activity下的Fragment可以获得同一个`ViewModel`,即可以交流。

比如下面的代码：

>Notice that both fragments use getActivity() when getting the ViewModelProvider. As a result, both fragments receive the same SharedViewModel instance, which is scoped to the activity.



```
public class SharedViewModel extends ViewModel {
    private final MutableLiveData<Item> selected = new MutableLiveData<Item>();

    public void select(Item item) {
        selected.setValue(item);
    }

    public LiveData<Item> getSelected() {
        return selected;
    }
}

public class MasterFragment extends Fragment {
    private SharedViewModel model;
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        model = ViewModelProviders.of(getActivity()).get(SharedViewModel.class);
        itemSelector.setOnClickListener(item -> {
            model.select(item);
        });
    }
}

public class DetailFragment extends Fragment {
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        SharedViewModel model = ViewModelProviders.of(getActivity()).get(SharedViewModel.class);
        model.getSelected().observe(this, { item ->
           // Update the UI.
        });
    }
}
```


### 总结

通过上面的介绍，我们可以通过`Lifecycle、LiveData、ViewModel`这三个对象写出一个比较稳定的app架构:代码逻辑清晰,生命周期管理方便等优点。

Demo地址: https://github.com/SusionSuc/ArchitectureComponentSmale





