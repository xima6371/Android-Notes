---
title: 服务的前世今生
date: 2019-04-20 11:39:55
tags: Service
---

介绍Service的启动方式及区别，以及如何保活

<!-- more -->

1. Service启动方式

   - Start：Context.startService()，启动后可无限制运行，在stopSelf()或者Context.stopService()后停止。
   - Bind：Context.bindService()，应用组件和Service进行绑定，可进行交互，甚至进程间通信IPC一个Service可与多个组件进行绑定，当所有组件都unBindService()之后停止工作。

   

2. Service生命周期

   ![](https://github.com/hadyang/interview/raw/master/android/images/service_life.png)

   

3. Service分类

   - 本地服务：start启动的服务，在应用内进行工作，设置android:exported=false确保服务仅适用于本地应用程序
   - 远程服务：bind绑定的服务，扩展Binder在应用内工作，使用Messenger让接口跨不同的进程工作，AIDL使服务具备多线程处理能力，且线程安全。
   - 前台服务：在onCreate()中调用startforeground(int ,Notification)即可
   - 后台服务：一般情况下都是后台服务

   

4. Service优先级

   - 前台进程：
     - 有一个可交互的Activity
     - Service与可交互的Activity绑定，或在前台运行，或正在执行生命周期
     - BoardcastReceiver正在执行onReceive()
   - 可见进程
     - Activity可见但不可交互
     - Service与可见的Activity绑定
   - 服务进程

     - 有一个start运行的Service
   - 后台进程

     - 有一个不可见的Activity
   - 空进程

     - 进程中不含任何活跃的组件

     

**保活**

   1. 设置为前台服务

   2. 在onStartConmand中返回Flags：START_STICKY，则服务在调用onStartConmand后被系统kill后会尝试重新启动服务。(进程被kill后就重建不了了)

   3. 在onDestory()中发送广播重新启动服务

   

