---
title: Android-ViewModel 使用指北
date: 2018-11-26
tag: Android
---

`Android` 开发近年越来越趋于成熟稳定，很多第三方组件也越来越完善，上半年 `Google` 爸爸发布了名为 `Jetpack` 的工具包堪称应用开发者的福音，这里就讨论一下这个开发套件中 `ViewModel` 到底是个什么样的存在。

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
<img src="https://developer.android.com/images/topic/libraries/architecture/viewmodel-lifecycle.png" alt="Illustrates the lifecycle of a ViewModel as an activity changes state." />
<img src="https://developer.android.com/images/topic/libraries/architecture/viewmodel-loader.png" />
<img src="https://developer.android.com/images/topic/libraries/architecture/viewmodel-replace-loader.png" />
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
class SimpleModel : ViewModel() {

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
class SimpleActivity : AppCompatActivity() {

    val viewModel by lazy { ViewModelProviders.of(activity).get(SimpleModel::class.java) }
    …
}
```

这段代码在需要使用到 viewmodel 的时候通过 __ViewModelProfiders#of(FragmentActivity)__ 来创建一个 `ViewModelProfider`，然后调用 `get` 方法创建出所需要的 `model` 实例  
非常简单地代码
