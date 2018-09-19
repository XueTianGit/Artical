title: Read 《Android开发艺术探索》 10.27
date: 2016/10/27 18:27:44       
categories: Android
---


### Read 《Android开发艺术探索》 10.27



#### Activity

- 生命周期

  - onPause():由于onPause()必须先执行完，新的Activty的onResume)()才会执行。因此在这个方法中可以做一些存储数据、停止动画等工作，但是注意不能太耗时。

  - onStart()和onStop()是从Activity是否可见这个角度来回调的，onResume()和onPause()是从Activity是否位于前台这个角度来回调的。

  - 异常生命周期

    - 除用户正常操作以外，其他情况下导致声明周期方法被调用
    - 对于异常情况，系统会在onSaveInstanceState()保存Activity的状态，在onCreate()或者onRestoreInstanceStae()方法中我们可以拿到保存的状态，区别是
      - 只要onRestoreInstanceState()调用，它是在onStart()之后，并且保存状态的数据Bundle必定不为null
      - onCreate()方法中的Bundle是可能为空的，因此使用前必须判断。

  - 异常情况下，除了Avtivity可以保存其状态外，View也可以通过onSaveInstanceState 和 onRestoreInstanceState来保存自己的状态。比如TextView中用户输入的数据，ListView的滚动位置等。

  - 通过指定Activity的configChanges属性，我们可以防止Activity在一些情况下的重建。

  - Activity启动流程

    > 启动Activity的请求会由Instrumentation来处理，然后它通过Binder向AMS发请求，AMS内部维护着一个ActivityStack并负责栈nei的Activity的状态同步，AMS通过ActivityThread去同步Activity的状态并完成生命周期方法的调用。

- 启动模式

  - Activity任务栈

    - taskAffinity属性指定Activity任务栈的名字，默认activity的taskAffinity值继承自Application的taskAffinity。而Application默认taskAffinity为包名，所以A的taskAffinity为包名。

    - allowTaskReparenting用于配置是否允许该activity可以更换从属task，通常和taskAffinity连在一起使用，用于实现把一个应用程序的Activity移到另一个应用程序的Task中。

    - 如果allowTaskReparenting为true的话，会有下面这种情况

      > 比如有两个应用A和B，A启动了B的一个ActivityC，然后按Home键回到桌面，然后再单机B的左面图标，这个时候并不是启动了B的主Activity，而是重新现身了已经被应用A启动的Activity。
      >
      > 这是因为 C 发现自己的任务栈 B 存在， 自己迁移到 B的任务栈中了。

    - standard模式

      - 当我们用ApplicationContext去启动这种模式的Activity的时候会报错。

        > Calling startActivity from outside of an Activity context requires the FLAG_ACTIVITY_NEW_TASK flag.

        这是因为standard模式的Activity默认会进入启动它的Activity所属的任务栈中，但是由于非Activity类型的Conetext并没有所谓的任务栈。

        解决办法是：为待启动Activity指定FLAG_ACTIVITY_NEW_TASK标志位，这样启动的时候就会为它创建一个新的任务栈，实际这个Activity是以singleTask模式启动的。

    - singleTop

      - 栈顶复用模式，即Activity如果已经存在于栈顶，则不会再次调用，而是回调其onNewIntent方法。

    - 标志位 FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS

      > 具有这个标记的Activity不会出现在历史Activity的列表中，当某些情况下我们不希望用户通过历史列表会到我们的Activity的时候这个标记比较有用。

- IntentFilter的匹配规则  action category data

  - 通过对intentFilter的匹配，我们可以启动Activity，Broadcast，Service

  - action要求Intent中必须有一个action且必须能够和过滤规则中的某个action相同。

  - category要求Intent可以没有category，但是如果你一旦有category，不管有几个，每个都要能够和过滤规则中的任何一个category相同。

  - data的匹配规则和action类似

    ```java
    <scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]
    ```

#### Android IPC

- Android中的多进程

  > - 在Android中多进程是指一个应用中存在多个进程的情况。
  > - 在Android中使用多进程只有一中方法，那就是给四大组件在AndroidMenifest中指定android：process属性。
  > - 另一种非常规的多进程方法就是通过JNI在native层去fork一个新的进程。

  - 多进程会对程序造成的影响
    - 静态成员和单例模式完全失效
    - 线程同步机制完全失效
    - SharePreferences的可靠性下降
    - Application会多次创建  （运行在同一个进程中的组件是属于同一个虚拟机和同一个Application）

  > 可以这样理解同一个应用的多进程：它就相当于两个不同的应用采用了SharedUID的模式。

- Serializable与Parcelable

  - Serializable

    - 采用ObjectOutputStream和ObjectInputStream可以轻松实现对象的序列化和反序列化。

    - serialVersionUID是用来辅助序列化和反序列过程的，原则上序列化后的数据中serialVersionUID只有和当前类的serialVersionUID相同才能正常的被反序列化。

      > 如果不指定serialVersionUID， 会根据类的结构来决定反序列化是否成功。

  - Parcelable

    - 需要实现Parcelable接口
    - writeToParcel()方法完成了序列化，具体实现是通过Parcel中一系列的write方法来完成
    - 反序列化功能由CREATOR来完成，实际实现是通过Parcel的read方法完成
    - 内容描述由describeContents方法来完成。

- Binder

  > Binder是Android中的一个类，它实现IBinder接口，是Android中的一种跨进程通信方式。
  >
  > Android开发中Binder主要用在Service中，包括AIDL和Messager。Messenger底层其实是AILDL。

  - AIDL

    - 当我们新建一个AIDL时，SDK会自动为我们生产AIDL所对应的Binder 类。

    - AIDL的本质是系统为我们提供了一种快速实现Binder的工具。

      > 当我们新建一个AIDL文件，并在其中定义好服务端提供的接口后，系统会自动生成对应的 xxx.java类。

      - 例如 Ibook.aidl  —> Ibook.java.   Ibook类即成IInterface接口。
      - 所有可以在Binder中传输的接口都必须即成IInterface接口。
      - Ibook.中有一个内部类Stub，他就是一个Binder类。
      - Stub实现Ibook的接口，并实现了客户端对远程服务端的访问实现

    - 由于我们通过Binder才能访问服务端的服务，Binder实现对服务端服务访问用两只方式来实现

      - 如果客户端进程和服务端进程位于同一进程，则具体实现逻辑有Binder完成
      - 否则，则有Binder内部的Proxy内部类完成。

    - Binder主要负责传递我们调用服务端接口时的参数，并返回服务端的结果。

      > - 当客户端发起远程请求时，由于当前线程会被挂起直到服务端进程返回数据，因此尽量不要在UI线程中发起远程请求
      > - 由于服务端的Binder方法运行在Binder的线程中，所以Binder方法不管是否耗时都应该采用同步的方式去实现。e

    - 对于服务端的实现，我们只要实现ADIL这个文件生成的抽象类Binder， 即可使用Binder来完成客户端和服务端的通信了。

  - Binder的 linkToDeath和unlinkToDeath 方法

    > 使用这两个方法我们可以监听服务端的Binder的存活情况，如果服务端的Binder挂掉，即我们远程调用不成功，我们可以做一些什么。