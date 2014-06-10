---
layout: post
title:  "苹果应用不同状态的回调处理"
date:   2012-04-16 16:17:59
categories: 
- code 
tags:
- obj-c

---

**iphone 应用不同状态切换时回调的方法**

<pre>
 application:didFinishLaunching:WithOptions:

 applicationDidBecomeActive:

 applicationWillEnterForeground:

 applicationWillResignActive:

 applicationDidEnterBackground:

 applicationWillTerminate:

 application:didChangeStatusBarFrame:

 UIApplicationDidReceiveMemoryWarningNotification:
</pre>

**iphone 应用的几个状态**

* Not running:程序完全没有执行。
* Inactive:程序在前景执行但是停止接收event,通常是因为系统有其它任务必须立刻告知使用者。
* Active:程序在前景执行，且可以正常接收event。
* Background:通常程序在要求进入suspend前会短暂进入此状态，程序可以做些收尾工作(ios4新增)。
* Suspended:应用程序在背景但不会继续执行，此时依然留在主记忆中，因此重新唤起需要的资源较少。