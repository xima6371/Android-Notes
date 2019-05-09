---
title: Activity两三事
date: 2019-04-11 18:12:55
tags: Activity

---



介绍Activity生命周期和启动模式

<!-- more -->



1. **Activity生命周期描述**

   - **onCreate()**：Activity实例被创建，加载布局资源和初始化操作
   - **onStart()**：注册BoardReceiver监测UI变化，进入前台，可见但不可交互。
   - **onResume()**：可见且可交互
   - **onPause()**：存储持久性数据，停止动画，释放系统资源(广播接收器等)以及一些耗电耗费CPU的内容，避免耗时操作，影响用户体验，可见，不可交互。
   - **onStop()**：完全不可见，进行比较耗费cpu的资源操作
   - **onDestroy()**：调用finish()或者点击返回时回调，释放所有资源。

   
2. **Activity四种启动模式**

   - **Standard**：每次启动Activity都会创建一个新的实例
   - **SingleTop**：栈顶复用，当栈中已经有该模式且处于栈顶的Activity存在时，再次启动该Activity不会创建新的实例，而是通过onNewIntent()复用之前的Activity。适用于收qq微信信息的activity
   - **SingleTask**：栈内复用，任务栈中只能有一个该模式的Activity实例存在，当再次启动该Activity时，会将栈中位置处于该Activity之上的所有Activity出栈，并通过onNewIntent复用。适用于主页面
   - **SingleInstance**：创建该模式的Activity会将其压入一个单独的任务栈中，且该任务栈有且只能有一个，一种Activity。

