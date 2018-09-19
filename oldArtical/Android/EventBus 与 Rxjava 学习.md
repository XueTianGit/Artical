title: EventBus 与 Rxjava 学习
date: 2016/10/26 18:27:44       
categories: Android
---

## EventBus 与 Rxjava 学习

#### EventBus

- 特点

  - 官方解释说它是是一个发布 / 订阅的事件总线
  - 使用它我们可以
    - 很方便的完成事件之间的同步，以及异步线程交互。
    - 代码干净整洁，基本无耦合。
  - 如果想要完成以上任务，则我们在使用eventbus时必须遵循规范，即
    - 注册事件
    - 处理事件的方法与其参数 应按照规范形式来定义  
    - 注销事件

- 使用方式与实现原理

  - 使用方式

    - 注册事件  

      ```java
       EventBus.getDefault().register(this); ／／EventBus为单例模式
      ```

    - 发布事件

      ```java
      EventBus.getDefault().post(EventType);
      ```

    - 按照规范，定义处理事件的方法 （eventbus新版本支持使用注解来定义处理事件的方法）

      ```java
       public void onEventMainThread(EventType event)  
      ```

    - 注销事件

      ```java
         EventBus.getDefault().unregister(this); 
      ```

  - 实现原理

    - 注册事件

      > - EventBus内部维护着一个Map集合，这个集合以“事件方法的参数的类”为key， 以含有这些key的类的集合为value。
      > - 注册事件是，就是 首先把事件和其相关参数（所属类、优先级、sticky）封装成一个subscribe，并添加到EventBus的内部集合中。

    - 发布事件

      > - 根据事件类型（就是处理事件方法的参数类型）去EventBus内部维护的集合，取出所有含有处理这个事件的类。
      > - 根据 事件应处理的线程、优先级、sticky，然后利用反射去含有对该事件类型处理的方法的类中调用处理事件的方法。

    - 处理事件

      > - 在发布事件时被反射调用， 从而完成“订阅”。

    - 注销事件

      > - 维护EnevtBus内部Map集合
      > - 即把该事件的事件类型所对应的类集合中，在类集合中移除当前类。



### RxJava

> 文章基于：http://gank.io/post/560e15be2dca930e00da1083
>

- 特点

  - 官方介绍：它是一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库。
  - 大神的理解：它就是一个实现异步操作的库，写出的代码十分的简洁。

- Observable : 决定什么时候触发事件以及触发怎样的事件

  - 创建方式

    ```java
    Observable observable = Observable.create(new Observable.OnSubscribe<String>() {
        @Override
        public void call(Subscriber<? super String> subscriber) {
            subscriber.onNext("Hello");
            subscriber.onNext("Hi");
            subscriber.onNext("Aloha");
            subscriber.onCompleted();
        }
    });

    OnSubscribe 会被存储在返回的 Observable 对象中，它的作用相当于一个计划表，当 Observable 被订阅的时候，OnSubscribe 的 call() 方法会自动被调用，事件序列就会依照设定依次触发
    ```


-   RxJava 提供了一些方法用来快捷创建事件队列

    ```java
    Observable observable = Observable.just("Hello", "Hi", "Aloha");
      
    String[] words = {"Hello", "Hi", "Aloha"};
    Observable observable = Observable.from(words);

    可以拆分集合，并将拆分后的数据依次发送到事件序列，即依次调用
    onNext("Hello"); onNext("Hi");  onNext("Aloha");   onCompleted();
    ```


- Observer : 决定事件触发的时候将有怎样的行为

  - 创建方式

    ```java
    Observer<String> observer = new Observer<String>() {
        @Override
        public void onNext(String s) {
            Log.d(tag, "Item: " + s);
        }

        @Override
        public void onCompleted() {
            Log.d(tag, "Completed!");
        }

        @Override
        public void onError(Throwable e) {
            Log.d(tag, "Error!");
        }
    };

     观察 String 类型的事件
    ```


-   Subscriber是Observer的子类，但其增加了两个方法

    > 1. `onStart()`: 这是 `Subscriber` 增加的方法。它会在 subscribe 刚开始，而事件还未发送之前被调用，可以用于做一些准备工作，例如数据的清零或重置。这是一个可选方法，默认情况下它的实现为空。需要注意的是，如果对准备工作的线程有要求（例如弹出一个显示进度的对话框，这必须在主线程执行）， `onStart()` 就不适用了，因为它总是在 subscribe 所发生的线程被调用，而不能指定线程。要在指定的线程来做准备工作，可以使用 `doOnSubscribe()` 方法，具体可以在后面的文中看到。
    > 2. `unsubscribe()`: 这是 `Subscriber` 所实现的另一个接口 `Subscription` 的方法，用于取消订阅。在这个方法被调用后，`Subscriber`将不再接收事件。一般在这个方法调用前，可以使用 `isUnsubscribed()` 先判断一下状态。 `unsubscribe()` 这个方法很重要，因为在 `subscribe()` 之后， `Observable` 会持有 `Subscriber` 的引用，这个引用如果不能及时被释放，将有内存泄露的风险。所以最好保持一个原则：要在不再使用的时候尽快在合适的地方（例如 `onPause()` `onStop()` 等方法中）调用 `unsubscribe()` 来解除引用关系，以避免内存泄露的发生。

-   除了Subscriber，RxJava还为Observable提供了许多不完全的回调

    ```java
          Action1<String> onNextAction = new Action1<String>() {
              // onNext()
              @Override
              public void call(String s) {
                  Log.d(tag, s);
              }
          };
          Action1<Throwable> onErrorAction = new Action1<Throwable>() {
              // onError()
              @Override
              public void call(Throwable throwable) {
                  // Error handling
              }
          };
          Action0 onCompletedAction = new Action0() {
              // onCompleted()
              @Override
              public void call() {
                  Log.d(tag, "completed");
              }
          };

          // 自动创建 Subscriber ，并使用 onNextAction 来定义 onNext()
          observable.subscribe(onNextAction);
          // 自动创建 Subscriber ，并使用 onNextAction 和 onErrorAction 来定义 onNext() 和 onError()
          observable.subscribe(onNextAction, onErrorAction);
          // 自动创建 Subscriber ，并使用 onNextAction、 onErrorAction 和 onCompletedAction 来定义 onNext()、 onError() 和 onCompleted()
          observable.subscribe(onNextAction, onErrorAction, onCompletedAction);

    ```

-   订阅

    - 使Subscribe和Observable发生关系，并启动整个事件序列


- 变换

  - 将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列

  - map：把事件对象的直接变换为另一种事件对象

    ```java
    Observable.just("images/logo.png") // 输入类型 String
        .map(new Func1<String, Bitmap>() {
            @Override
            public Bitmap call(String filePath) { // 参数类型 String
                return getBitmapFromPath(filePath); // 返回类型 Bitmap
            }
        })
        .subscribe(new Action1<Bitmap>() {
            @Override
            public void call(Bitmap bitmap) { // 参数类型 Bitmap
                showBitmap(bitmap);
            }
        });

    Func1用来封装我们自定义的事件对象变换方式的方法。
    FuncX 和 ActionX 的区别在 FuncX 包装的是有返回值的方法。
    ```


-   flatMap：

    ```java
    Student[] students = ...;
    Subscriber<Course> subscriber = new Subscriber<Course>() {
        @Override
        public void onNext(Course course) {
            Log.d(tag, course.getName());
        }
        ...
    };
    Observable.from(students)
        .flatMap(new Func1<Student, Observable<Course>>() {
            @Override
            public Observable<Course> call(Student student) {
                return Observable.from(student.getCourses());
            }
        })
        .subscribe(subscriber);

    flatMap() 和 map() 有一个相同点：它也是把传入的参数转化之后返回另一个对象。但需要注意，和 map() 不同的是， flatMap() 中返回的是个 Observable 对象，并且这个 Observable 对象并不是被直接发送到了 Subscriber 的回调方法中。 flatMap() 的原理是这样的：1. 使用传入的事件对象创建一个 Observable 对象；2. 并不发送这个 Observable, 而是将它激活，于是它开始发送事件；3. 每一个创建出来的 Observable 发送的事件，都被汇入同一个 Observable ，而这个 Observable 负责将这些事件统一交给 Subscriber 的回调方法。这三个步骤，把事件拆成了两级，通过一组新创建的 Observable 将初始的对象『铺平』之后通过统一路径分发了去。而这个『铺平』就是 flatMap() 所谓的 flat。
    ```


- 线程变换

  - Scheduler ： 线程控制器，RxJava 通过它来指定每一段代码应该运行在什么样的线程。

    > - `Schedulers.immediate()`: 直接在当前线程运行，相当于不指定线程。这是默认的 `Scheduler`。
    > - `Schedulers.newThread()`: 总是启用新线程，并在新线程执行操作。
    > - `Schedulers.io()`: I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 `Scheduler`。行为模式和 `newThread()` 差不多，区别在于 `io()` 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 `io()` 比 `newThread()` 更有效率。不要把计算工作放在 `io()` 中，可以避免创建不必要的线程。
    > - `Schedulers.computation()`: 计算所使用的 `Scheduler`。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 `Scheduler` 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 `computation()` 中，否则 I/O 操作的等待时间会浪费 CPU。
    > - 另外， Android 还有一个专用的 `AndroidSchedulers.mainThread()`，它指定的操作将在 Android 主线程运行。

  - subscribeOn（）与 observeOn（）

    - subscribeOn 指定 subscribe() 发生在 IO 线程
    - observeOn 指定 Subscriber 的回调发生在主线程

    ```java
    Observable.just(1, 2, 3, 4) // IO 线程，由 subscribeOn() 指定
        .subscribeOn(Schedulers.io())
        .observeOn(Schedulers.newThread())
        .map(mapOperator) // 新线程，由 observeOn() 指定
        .observeOn(Schedulers.io())
        .map(mapOperator2) // IO 线程，由 observeOn() 指定
        .observeOn(AndroidSchedulers.mainThread) 
        .subscribe(subscriber);  // Android 主线程，由 observeOn() 指定
    ```

### RxJava 与 Retrfiot

> Retrofit 除了提供了传统的 `Callback` 形式的 API，还有 RxJava 版本的 `Observable` 形式 API。下面我用对比的方式来介绍 Retrofit 的 RxJava 版 API 和传统版本的区别。

-  RxJava 形式的 API

```java
@GET("/user")
public Observable<User> getUser(@Query("userId") String userId);


getUser(userId)
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Observer<User>() {
        @Override
        public void onNext(User user) {
            userView.setUser(user);
        }

        @Override
        public void onCompleted() {
        }

        @Override
        public void onError(Throwable error) {
            // Error handling
            ...
        }
    });


两层网络请求的写法：
  
@GET("/token")
public Observable<String> getToken();

@GET("/user")
public Observable<User> getUser(@Query("token") String token, @Query("userId") String userId);

getToken()
    .flatMap(new Func1<String, Observable<User>>() {
        @Override
        public Observable<User> onNext(String token) {
            return getUser(token, userId);
        })
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Observer<User>() {
        @Override
        public void onNext(User user) {
            userView.setUser(user);
        }

        @Override
        public void onCompleted() {
        }

        @Override
        public void onError(Throwable error) {
            // Error handling
            ...
        }
    });

  
```