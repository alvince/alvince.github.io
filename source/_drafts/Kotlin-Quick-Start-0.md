---
title: Kotlin 快速入门手册-0
date: 2019-02-13
tags: [Kotlin, Tutorials]
---

9012 年的今天  
依然还有这么多项目跟团队依然在使用着臃肿冗余的 Java 开发着 Android 应用  
简直是听者伤心闻者落泪 …  
<!-- more -->

## 0 - Kotlin 跟 Java 的区别

首先 `Kotlin` 跟 `Scala` 一样都是从 `Java` 扩展出的一种 `JVM` 语言，这个决定了其运行上跟 `Java` 的兼容性  

`Kotlin` 在设计上是为了解决 `Java` 语言使用上的一些痛点，比如空指针 …  
另外 `Kotlin` 摒弃了 `Java` 中的基本类型，在语言层面屏蔽了 `int` `long` `boolean` 等这些内置基本类型，全部使用 `Kotlin` 内置类型替代了。  

同时，`Kotlin` 引入了一些其他高级语言的特性，内置原生 __lambda__ 支持，函数式编程支持，属性代理，操作符重载等  
与 `Java` 不同，`Kotlin` 中变量、函数上升为顶级成员，不再需要为所有的代码加上类定义，原生支持了函数式编程写法

## 1 - 数据类型 & 类型

#### 数字

使用 `Kotlin` 内置的如 _Int Boolean Char String Array_ 等类型代替了 `Java` 中的基本数据类型，参阅 `kotlin.*`  

`Kotlin` 中的数字类型都继承自 `Number`，内置一键转换成其他类型的函数: _toDouble, toFloat, toXxx 等_  

同时内置了很多用于数据计算的函数，如: plus, minus, times, div

#### 字符 & 字符串



#### 类型和类


## 1 - 函数

`Kotlin` 原生支持函数式编程，函数(function) 作为顶级成员可直接访问，且函数可作为一类特殊的对象传递给其他函数（参考 JavaScript 闭包）  

`Kotlin` 中函数（方法）使用关键词 __fun__ 声明，语法结构为 [modifer] fun name([args])[:Type] {}  
`Unit` 意为无类型，类似 `Java` 中的 `void`

```Kotlin
protected fun testFunc(args: Int): Unit {
    ...
}
```

## 2 - 基本语法
