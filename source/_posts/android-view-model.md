---
title: Android-ViewModel 使用指北
date: 2018-11-26
tag: Android
---

`Android` 开发近年越来越趋于成熟稳定，很多第三方组件也越来越完善，上半年 `Google` 爸爸发布了名为 `Jetpack` 的工具包堪称应用开发者的福音，这里就讨论一下这个开发套件中 `ViewModel` 到底是个什么样的存在。
<!-- more -->

## 什么是 Jetpack

先说一下 [Jetpack](https://developer.android.com/jetpack/)，官方对此介绍是帮助开发者专注应用的业务代码、更轻松地开发出色的应用程序的一个 Android 软件开发组件集合  
> Jetpack is a collection of Android software components to make it easier for you to develop great Android apps. 
> These components help you follow best practices, free you from writing boilerplate code, and simplify complex tasks, so you can focus on the code you care about.

里面包含了几乎所有开发 Android App 能用到的涉及程序架构、导航、UI、行为相关的组件库

- Foundation  
  `AppCompat`, `Android KTX`, `Multidex`, `Test`
- Architecture  
  `Data Binding`, `Lifecycles`, `LiveData`, `Navigation`, `Paging`, `Room`, `ViewModel`, `WorkManager`
- Behavior  
  `Download manager`, `Media & playback`, `Notifications`, `Permissions`, `Preferences`, `Sharing`, `Slices`
- UI  
  `Animation & transitions`, `Auto`, `Emoji`, `Fragment`, `Layout`, `Palette`, `TV`, `Wear OS by Google`

在这里就来分析一下 `ViewModel` 到底应该怎么使用

## Android 应用程序架构

讨论 ViewModel 之前需要先弄清楚一个问题，那就是通常在 Android App 开发中使用软件应用程序架构。  
Google 爸爸同样发布了一系列软件架构的示例参考 👉  [android-architecture](https://github.com/googlesamples/android-architecture)

在 8012 年的今天，__MVC__ 这种上古时代的移动软件架构应该已经淘汰的差不多了，先说一下 __MVP__ 这个现在几乎软件必备的软件架构。

#### MVP

所谓 `MVP` 就是把程序代码 按照 __model-view-presenter__ 分层解耦  
`Presenter` 通常是通过对 网络层调用、数据解析、业务处理以及其他逻辑代码的封装  
然后通过对 `View` 层接口的调用来通知 `UI` 界面更新  
通过将 __数据__、__视图__、__逻辑__ 三层代码隔离来达到解耦的目的  
但是这种架构有个问题，那就是 `Presenter` 非常依赖 `View` 层的接口  

这种模式下，逻辑-视图之间的通讯完全依赖于 View 层的接口，一旦业务发生了变更，需要修改 View 层接口的声明、Presenter 对 View 通知调用的入口、以及 `Activity` / `Fragment` 等视图接口的实现  

#### MVVM

那么在这种情况下 `Android` 开发的关注焦点转移到了 `MVVM` 架构模式  
这种模式其实是对 `MVC` 的另一层变种，这里的 `VM` 指的就是 `ViewModel`（跟下面讨论的 Architecture ViewModel 不一样）  
区别于 `MVP` 对业务逻辑的完全抽离，`MVVM` 在视图之上抽象出了一个 `ViewModel` 的概念，这个层面关注了界面视图所关联的 __数据 + 逻辑__ 的操作  

有过网页前端或者 iOS 相关开发经验的同学应该能够了解，其实就是 __Data Binding__ 这种开发模式  
通过将数据绑定到视图层，更新相应的数据 Model 来触发UI视图的状态更新  
一个典型的场景就是基于 `React` 环境的开发，通过改变组件的 `props` 和 `state` 来通知 UI 视图刷新显示内容

关于 `MVVM` 的更多内容这里不展开细说，下面开始讨论一下 `Android Jetpack` 架构组件中 `ViewModel` 的使用

## ViewModel

[ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel) 这个组件是什么？

官方介绍是这样的：
> The ViewModel class is designed to store and manage UI-related data in a lifecycle conscious way. 
> The ViewModel class allows data to survive configuration changes such as screen rotations.

这里明确说明了 ViewModel 是出于管理和存储 UI 相关数据的目的，通过对 Android 组件生命周期感知的方式而设计出来的一个组件，并且允许在设备的配置发生变更（如屏幕旋转）时继续保持。

这样的话最直接的好处就是开发者不再需要关心 `Activity` / `Fragment` 的相关状态数据持久化（saveInstanceStatus）的问题了，能够专注于产品业务的开发，避免缓存恢复数据这种重复模板化操作。

这是官方文档上对 ViewModel 如何工作的示意图 👇

<div style="background:white">
<img src="http://pic.ffsky.net/images/2018/11/27/9ad50f26af937fd6731d13f327b1a930.png" alt="Illustrates the lifecycle of a ViewModel as an activity changes state." />
<img src="http://pic.ffsky.net/images/2018/11/27/e2367f4091012a8eeaf42352dceee7c9.png" alt="viewmodel-loader" />
<img src="http://pic.ffsky.net/images/2018/11/27/23294c647338dbebd78a55684320f56c.png" alt="viewmodel-replace-loader" />
</div>
<p>

可以看到 `ViewModel` 跟随 `Activity` 创建之后会一直在在内存中生存并且工作  
直到与其关联的 `Activity` 销毁 `destroy` 时，会触发一个叫 `onCleared` 的生命周期函数，最后宣告死亡。

__ViewModel__ 的生命周期感知依赖于 `Jetpack` 的另一个组件 [Lifecycles](https://developer.android.com/topic/libraries/architecture/lifecycle)
关于 `Lifecycles` 这里不详细展开，只需明确 `Lifecycles` 通过对 `Activity` 生命周期的捕获来触发生命周期事件的上报来达到监控生命周期的目的

基于 `Lifecycles` 意味着开发者不再需要手动关注组件的生命周期问题，那么 `ViewModel` 是如何感知并且同步应用组件的生命周期呢？

通过查看 `ViewModel` 源代码可以看出，它本身其实就是一个普通的 Java Bean，没有任何复杂的地方  
```Java
public abstract class ViewModel {
    /**
     * This method will be called when this ViewModel is no longer used and will be destroyed.
     * <p>
     * It is useful when ViewModel observes some data and you need to clear this subscription to
     * prevent a leak of this ViewModel.
     */
    @SuppressWarnings("WeakerAccess")
    protected void onCleared() {
    }
}
```
那么问题来了，这样只怎么实现数据和生命周期的管理呢

<div>
<img src="http://pic.ffsky.net/images/2018/11/26/7d5e164647c71a88dca23044abe0242a.png" alt="7d5e164647c71a88dca23044abe0242a.png" style="margin-left:0">
</div>

翻看 viewmodel 的源代码会发现这个库实在是简单到了极点，只有几个类和接口，这就是为什么要使用 `Jetpack` 组件是应该依赖 `AppCompat` 兼容套件了  
因为 `Jetpack` 实现生命周期相关的组件其实是基于 __support-v4__ 来实现，通过 `FragmentActivity` 来实现组件相关的接口来完成对 `ViewModel` 的管理的  
至于具体是怎么实现这个管理过程，这里暂不细讲，有兴趣可以自行阅读 `FragmentActivity` 的源码

#### 创建 ViewModel

要使用 `ViewModel` 来管理数据，首先需要声明一个 model 类来集成 __android.arch.lifecycle.ViewModel__
```Kotlin
class SampleModel : ViewModel() {

    var profile: UserInfo? = null

    override fun onCleared() {
        profile = null
    }
}
```

回到正题，实际开发中要怎么来使用 ViewModel 呢？
库中提供了一个 `ViewModelProviders` 的工具类来帮助创建 `ViewModel `

<div style="align:left">
<img src="http://pic.ffsky.net/images/2018/11/26/a9e53786ae08929990cafda692b4b330.png" alt="a9e53786ae08929990cafda692b4b330.png" style="margin-left:0">
</div>

```Kotlin
class SampleActivity : AppCompatActivity() {

    val viewModel by lazy { ViewModelProviders.of(activity).get(SimpleModel::class.java) }
    …
}
```

这段代码在需要使用到 viewmodel 的时候通过 __ViewModelProfiders#of(FragmentActivity)__ 来创建一个 `ViewModelProvider`，然后调用 `get` 方法创建出所需要的 `model` 实例  
非常简单地代码，到这里 viewmodel 看上去好像已经说完了  
…  
盐鹅怎么可能呢，讲到这里好像跟 `MVVM` 还是没什么太大的关系  
那么这里需要介绍 __Jetpack Architecture__ 的另一个重要的组件 __LiveData__ 了  

## LiveData

还是先看官方说明
> LiveData is an observable data holder class. Unlike a regular observable, LiveData is lifecycle-aware, meaning it respects the lifecycle of other app components, such as activities, fragments, or services. 
> This awareness ensures LiveData only updates app component observers that are in an active lifecycle state.

这里已经清晰的介绍了 LiveData 是一个可观察的数据持有类，而且跟普通的观察者不一样的地方在于  
LiveData 可以感知 Android 系统组件生命周期，并且只在观察者组件处于活跃状态的时候才通知更新  

看到这里是不是就想起来前面说的数据绑定操作？
没错，这样代码就变成了介个样子
```Kotlin
class SampleModel : ViewModel() {
    val profileData = MutableLiveData<UserInfo>()

    var profile: UserInfo? = null

    override fun onCleared() {
        profile = null
    }

    fun fetchUserInfo() {
        …
        // 网络或本地持久化数据获取用户信息
        …
        // 获取成功后使用 LiveData 通知订阅端
        profileData.postValue(profile)
    }
}

class SampleActivity : AppCompatActivity() {

    val viewModel by lazy { ViewModelProviders.of(activity).get(SampleModel::class.java) }
    …

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        …
        viewModel.profileData.observe(this, Observer {
            it?.also {
                // it -> 接收到的 UserInfo
                …
                …
                /* 更新 UI */
            }
        })
    }
}
```

这样一来，`ViewModel` 只需要扮演数据持有者以及相关业务控制器，不用像 __Presenter__ 那样需要关心 __View__ 层的接口变化了，而且能够有效避免因 __Presenter__ 持有 __View__ 引用造成的内存泄漏问题

这里简单看一下 `LiveData` 的代码
```Java
/**
     * Posts a task to a main thread to set the given value. So if you have a following code
     * executed in the main thread:
     * <pre class="prettyprint">
     * liveData.postValue("a");
     * liveData.setValue("b");
     * </pre>
     * The value "b" would be set at first and later the main thread would override it with
     * the value "a".
     * <p>
     * If you called this method multiple times before a main thread executed a posted task, only
     * the last value would be dispatched.
     *
     * @param value The new value
     */
    protected void postValue(T value) {
        boolean postTask;
        synchronized (mDataLock) {
            postTask = mPendingData == NOT_SET;
            mPendingData = value;
        }
        if (!postTask) {
            return;
        }
        ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
    }

    /**
     * Sets the value. If there are active observers, the value will be dispatched to them.
     * <p>
     * This method must be called from the main thread. If you need set a value from a background
     * thread, you can use {@link #postValue(Object)}
     *
     * @param value The new value
     */
    @MainThread
    protected void setValue(T value) {
        assertMainThread("setValue");
        mVersion++;
        mData = value;
        dispatchingValue(null);
    }
```
可以看出 LiveData 通过设置 value 来驱动数据变化事件，内部通过分发新的 data 通知到订阅者（通常是UI界面的数据渲染代码）  
这里有两个相关的方法：`setValue` 以及 `postValue`  
查看 `postValue` 方法最后一行可以看出通过 post 来改变数据时，内部会通过一个任务调度器来事件分发到主线程，  
而 `setValue` 方法通过注解声明以及首行的断言检查可知，此方法需要在主线程中调用，这意味着订阅的 Observer __始终会在主线程中响应__

这两个方法的可见性修饰是 `protected`，意味着开发者不能直接调用，所以 __LiveData__ 提供了一个子类 `MutableLiveData`
```Java
/**
 * {@link LiveData} which publicly exposes {@link #setValue(T)} and {@link #postValue(T)} method.
 *
 * @param <T> The type of data hold by this instance
 */
public class MutableLiveData<T> extends LiveData<T> {
    @Override
    public void postValue(T value) {
        super.postValue(value);
    }

    @Override
    public void setValue(T value) {
        super.setValue(value);
    }
}
```
代码非常简单，只是单纯的修改了这个两个更新数据的方法的修饰，暴露给了开发者调用  
当我们需要改变数据的时候只需要调用一下 `postValue` 方法就可以通知视图订阅来更新视图显示的内容和状态了，就像 __React__ 中 对 `props` 和 `stat` 赋值一样简单

所以 … 赶紧开始拥抱、以及享受 __ViewModel__ + __LiveData__ 带来的便捷吧
