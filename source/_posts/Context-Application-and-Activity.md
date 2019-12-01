---
title: 「Android」Context，Application 和 Activity
date: 2019-12-01 10:22:18
tags:
	- Android
	- Context
	- Application
	- Activity
categories:
	- Android
---



**简述**：`Context` 在 `Android` 开发中无处不在，可以说是最重要的概念。对 `Context` 的错误使用，很容易造成 `Android App` 的内存泄漏。

<!-- more -->

<br />

# 前言

1. `Application` 和 `Activity` 都**间接继承**自 `Context` 。
2. 所谓 `Application Context` ，就是指 `Application`。同样的，`Activity Context` 也就是指 `Activity`。
3. 本文把等同的事物（比如 `Activity Context` 和 `Activity`）当作不同来讲，逻辑上似乎有些奇怪，但这主要是为了强调它（`Activity Context`）一方面提供了应用级和系统级的接口，另一方面（`Activity`）也是显示的主体。

<br />



# [Context](https://developer.android.com/reference/android/content/Context#getApplicationContext())

```
java.lang.object
  ↳ android.content.Context
```

1. 是什么

   > Interface to global information about an application environment. This is an abstract class whose implementation is provided by the Android system. It allows access to application-specific resources and classes, as well as up-calls for application-level operations such as launching activities, broadcasting and receiving intents, etc. 
   >
   > `Context` 是访问应用程序全局信息的接口，是一个由`Android` 系统提供实现（类 `ContextImpl`）的抽象类。它允许访问应用程序特有的资源（`resource`）和类（`class`），以及向上调用（`up-call`）应用级操作，如启动 `Activity`、`Broadcast` 和接收 `Intent` 等。 
   
2. 能做什么

   - 创建 `View`（比如 `TextView` 、`Button`、`ImageView` 等）
   - 启动 `Activity`、`Service` 
   - 发送和接收广播（`Broadcast`）
   - 。。。。。。

3. 实现

   `Context` 只是一个抽象类，其实现是继承`Context` 的 类 `ContextImpl`。

<br />



# [Activity]( https://developer.android.com/reference/android/app/Activity ) 和 [Application]( https://developer.android.com/reference/android/app/Application )

```java
java.lang.object
  ↳ android.content.Context
      ↳ android.context.ContextWrapper
          ↳ android.view.ContextThemeWrapper
             *↳ android.app.Activity 
         *↳ android.app.Application
```

可以看出，`Activity` 和 `Application` 间接继承了 `Context`，那这两者各自有什么特点，之间又有什么区别呢？

## 特点

1. 当应用程序的进程启动时，第一个实例化的类就是 `Application`。`Application` 是一个单例（`singleton`）的类，即一个运行的应用只有 `Application` 或其子类的一个实例。
2. `Activity` 的基本任务是创建一个 Window，用于显示 调用方法 `setContentView(View)` 生成的 `UI`。

<br />

## 区别

1. 有不同的生命周期。`Application Context` 存在于应用程序的运行期间，而 `Activity Context` 则与 `Activity ` 的实例”同生共死“。

2. 有不完全相同的应用场景。

   |                                                       | Application | Activity |
   | ----------------------------------------------------- | :---------: | :------: |
   | 启动 Activity / Start an activity                     |      ✘      |    ✔     |
   | 显示对话框 / Show a dialog                            |      ✘      |    ✔     |
   | 生成布局 / Layout inflation                           |      ✘      |    ✔     |
   | 加载资源 / Load resource values                       |      ✔      |    ✔     |
   | 启动 Service / Start a service                        |      ✔      |    ✔     |
   | 与 Service 绑定 / Bild to a service                   |      ✔      |    ✔     |
   | 发送 Broadcast / Send a broadcast                     |      ✔      |    ✔     |
   | 注册 Broadcast 接收器 / Register a broadcast receiver |      ✔      |    ✔     |

   显然，`Activity Context` 能做到的比 `Application Context` 更多。

<br />



# 获取 Context

## 方法

1. [`Context.getApplicationContext()`]( https://developer.android.com/reference/android/content/Context#getApplicationContext() )：返回当前进程的唯一、全局的 `Application Context`，它的生命周期与当前 `Context` 并不相干。

2. [`Activity.getApplication()`]( https://developer.android.com/reference/android/app/Activity#getApplication() ):  返回该 `Activity` 的 `Application` ，和 `getApplicationContext()` 返回的是同一个对象。

3. [`View.getContext()`](https://developer.android.com/reference/android/view/View#getContext())：返回一个 `Context` 引用，指向 `View` 运行依靠的 `Context`，通常就是当前正在显示的 `Activity` 实例。

4. `Activity.this`：适用于在 `Activity` 内的匿名内部类 访问 其外部的 `Activity`。

5. [`ContextWrapper.getBaseContext()`](https://stackoverflow.com/questions/9605459/android-why-must-use-getbasecontext-instead-of-this)：返回 `base Context`。

   

## 选用

1. 避免使用 `getBaseContext()`。因为你并不知道返回的是哪个`Context`。
2. 与 `UI` 相关的场景，使用 `Activity Context`；除此之外，可以使用 `Application Context`。
3. 将一个 `Context` 传到 `Activity` 的范围之外，选择 `Application Context` 能够很简单地避免内存泄漏，减少很多考虑。
4. 最重要的一点，确保不要将短暂存在的 `Context` 传递给长时间存在的对象，比如将 `Activity Context` 传给一个长时间在后台运行的 `Service`。

<br />



# 参考

- [Context - Android](https://developer.android.com/reference/android/content/Context#getApplicationContext())
- [Application - Android]( https://developer.android.com/reference/android/app/Application )
- [Activity - Android](https://developer.android.com/reference/android/app/Activity)
- [View - Android](https://developer.android.com/reference/android/view/View)
- [Android 中用 getApplicationContext() 会不会避免某些内存泄漏问题 - ZhiHu](https://www.zhihu.com/question/34007989) 
- [如何理解Context? - ZhiHu]( https://zhuanlan.zhihu.com/p/27163977 )
- [Which Context should I use in Android? - Medium]( https://medium.com/@ali.muzaffar/which-context-should-i-use-in-android-e3133d00772c )
- [Understanding Context In Android Application - Mindorks]( https://blog.mindorks.com/understanding-context-in-android-application-330913e32514 )
- [Mastering Android context - FreeCodeCamp](https://www.freecodecamp.org/news/mastering-android-context-7055c8478a22/ )
- [Using Context - CodePath](https://guides.codepath.com/android/Using-Context#avoiding-memory-leaks) 
- [Using Application context everywhere? - StackOverFlow](https://stackoverflow.com/questions/987072/using-application-context-everywhere)
- [Android: why must use getBaseContext() instead of this](https://stackoverflow.com/questions/9605459/android-why-must-use-getbasecontext-instead-of-this)

<br />





**以上！**

<br />