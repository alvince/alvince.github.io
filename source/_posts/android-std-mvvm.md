---
title: 关于标准 MVVM 设计模式在 Android 中应用的思考
date: 2021-09-06 18:27:12
categories:
- Android
tags:
- Android
- Jetpack
- MVVM
---

本来这篇文章很早就应该写的，一直没(比)有(较)时(懒)间  
今天决定把它写完咯

> 首先表明态度, I think:  
> 网上流传的 `ViewModel` + `LiveData` + XXX 的号称 `MVVM` 的代码设计基本都是假的（fake news :P)

<!-- more -->

MVVM
---

首先说一下 `MVVM` 这种架构模式（或者说设计模式），我对它的认识源于维基百科  

![mvvm_pattern](wiki_mvvm_pattern.png)

我们来看看关于 `MVVM` 各组件的标准定义

- __`model`__: 没啥好说的，跟 `MVP` `MVC` 之流大差不差，定义了业务（数据）逻辑和数据的模型
- __`view`__: 还是没啥好说的，就是视图层，用户界面
- __`view-model`__: 关键在这里，我们所谓的 `view-model` 其实是从 `presenter` 胶水演化出来的，暴露视图的公开属性和指令，本质上就是 `view` 视图对应的一个 `model`  
  - 首先，它是一个 view model，对 `model` 层暴露来自 `view` 层的一些公开属性，以及来自 `view` 层的一些指令（presenter?）
  - And，__binder__:
    `view-model` 区别于 `presenter` 与 `controller`，它有一个称之 `binder` 的东西，作用是处理 view-model 中暴露的视图属性（状态）与视图 UI 的自动同步

所以 MVVM 模式下的流转路径应该是这样的：
- `view`: user input event -> `view-model`: view property or command
- `view-model`: handle input, biz model -> `model`: biz data/logic processing
- `model`: state chagne events(data) -> `view-model`: handle biz state
- `view-model`: ui state update -> `binder`: handle ui state synching

MVP
---

ok，我们再来看一下 `MVP` 是怎么流转的

![mvp_mode](mvp_mode.png)

- `view`: User input event -> `presenter`: function (command)
- `presenter`: biz model -> `model`: biz data/logic processing
- `model`: state change events(data) -> `presenter`: convert biz state
- `presenter`: biz state/data -> `view`: refresh ui

仔细对比一下往上流传的基于 `Jetpack` `ViewModel` + `LiveData` 的伪 `MVVM`（MVP）的流转情况：

1. `Activity`/`Fragemnt` -> `ViewModel`, 这是 `view` -> `presenter`
2. `ViewModel` 调用 `model` 处理网络请求、数据逻辑、文件 IO 等业务逻辑，这是 `presenter` -> `model`
3. 这里我们看一下在 `ViewModel` 中完成了业务逻辑通知 _UI_ 刷新，通过 `LiveData` 的 `setValue`/`postValue` 更新状态，  
view 层通过 viewModel.xxxLiveData.observe(lifecycleOwner) { data -> … }

在最后一个环节，我们对比一下经典的 `MVP` 模式的写法：`presenter` 通过持有的 `view` 接口通知视图变更，`view` 层在对应的接口实现中完成对 _UI_ 组件的更新

看 ~ 发现了什么  
即使是通过 `LiveData` 观察者模式在 `view` 层实现对数据的观察，省去了经典 `MVP` 写法的 `view` 接口定义和耦合，但是在事件（数据）流转的路径上，依然是走的 `MVP` 的模式  

对比 `MVVM` 定义的工作流程，不难发现，其中最大的差异在于 `binder` 这个角色的存在  
__binder__ 作为实现数据和 _UI_ 同步的重要组件，同时按照 `MVVM` 模式的定义，属于 `view-model` 的内部成员  

因此可以得出结论：`MVVM` 的关键在于，用户事件的流转是单向，从 `view` 层开始，到 `view-model` 结束；而这其中的关键在于 __binder__

Jetpack data-binding
---

[Databinding](https://developer.android.com/jetpack/androidx/releases/databinding) 就是 `Google` 爸爸为我们提供的一个官方 `binder` 实现方案

即：
- `MVVM` 中的 `binder` 可以直接使用官方 `data-binding` 组件来实现
- 不使用 `data-binding` 组件，自定义处理数据与 _UI_ 的同步的 `binder` 并在 `view-model` 中维护，也是规范的 MVVM 写法

__DataBinding__ 分为两个部分

`ViewDataBinding`：每个被 `<layout>` 标签包裹的布局都会对应生成一个 `ViewDataBinding` 的子类作为视图与数据绑定的管理者
`Observable`/`BaseObservable`: 实际的 `binder` 开发接口，需要绑定与 `view` 层建立绑定关系的数据通过实现此接口并注册成员，即可自动完成监听与同步（实际代码在生成的 XxxLayoutBindingImpl 类中）

关于 `DataBinding` 的使用及原理，此处不予赘述

MVVM 在实际项目中的落地
---

#### sample Activity

```kotlin
class SampleBindingActivity : AppCompatActivity(), ActivityBindingHolder<SampleActivityBinding> by ActivityBinding(R.layout.sample_activity) {

    private val viewModel: SampleViewModel by viewModel()

    override fun onCreate(savedInstanceState: Bundle?) {
        // replace setContentView(), and hold binding instance
        inflateBinding { binding ->
            // init with binding
            binding.initView()
            …
            viewModel.bind(binding)
        }
        …
    }

    private fun SampleActivityBinding.initView() {
        val random = Random()
        btnTest.onClick {
            viewModel.random = random.nextInt(100)
        }
    }
}
```

#### sample layout

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable
            name="binder"
            type="package.SampleBinder" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <androidx.appcompat.widget.AppCompatTextView
            android:id="@+id/tv_nickname"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:maxLines="1"
            android:text="@{binder.nickname}"
            android:textSize="12sp"
            app:layout_constraintBottom_toTopOf="parent"
            app:layout_constraintEnd_toEndOf="@id/channel_name"
            app:layout_constraintStart_toStartOf="@id/program_vip" />

        <androidx.appcompat.widget.AppCompatButton
            android:id="@+id/btn_test"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/tv_nickname" />
    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

#### sample view-model

```kotlin
class SampleViewModel : ViewModel() {

    var random: Int by ObservableProperty(0) { binder.nickname = "Nickname_$it" }

    private val model = SampleModel() // biz handler model, network/data/io etc.
    private val binder = SampleBinder() // binder for sync data and view state

    fun bind(binding: ViewDataBinding) {
        binding.setVariable(BR.binder, binder)
    }

    …
}
```

#### sample binder

```kotlin
class SampleBinder : BaseObservable() {

    @get:Bindable
    var nickname: String by observableField(BR.nickname, "Nickname")

    …
}
```

注：
---

`ActivityBindingHolder` 👉🏻 [android-zanpakuto: ActivityBinding.kt](https://github.com/alvince/android-zanpakuto/blob/main/databinding/src/main/java/cn/alvince/zanpakuto/databinding/ActivityBinding.kt)  
func: `viewModel` 👉🏻 [android-zanpakuto: ViewModel.kt](https://github.com/alvince/android-zanpakuto/blob/main/lifecycle/src/main/java/cn/alvince/zanpakuto/lifecycle/ViewModel.kt)
func: `ObservableProperty` 👉🏻 [android-zanpakuto: ObservableProperty.kt](https://github.com/alvince/android-zanpakuto/blob/main/core/src/main/java/cn/alvince/zanpakuto/core/property/ObservableProperty.kt)
func: `observableField` 👉🏻 [android-zanpakuto: ObservableAttribute.kt](https://github.com/alvince/android-zanpakuto/blob/main/databinding/src/main/java/cn/alvince/zanpakuto/databinding/property/ObservableAttribute.kt)
