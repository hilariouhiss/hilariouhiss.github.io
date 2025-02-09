---
title: 安卓开发基础
date: 2025-01-19
slug: android-dev-basics
categories:
  - Android
description: 安卓开发基础知识
draft: true
lastmod: 2025-01-19
---
## 关键类和方法

### Activity

Activity 是 Android 应用中最基本的界面单元，Activity 具有自己的生命周期，并由一系列回调函数表示不同的状态。

- 创建状态：onCreate，表示 Activity 正在被创建，系统调用 `onCreate()` 方法进行初始化设置，如加载布局、初始化成员变量等
- 回复状态：onStart，表示 Activity 对用户可见，准备开始交互
- 活跃状态：onResume，表示 Activity 处于前台，具有用户的输入焦点
- 暂停状态：onPause，系统即将启动或回复另一个 Activity
- 停止状态：onStop，Activity 对用户不再可见，将释放部分资源，但仍存在于内存中
- 销毁状态：onDestroy，Activity 被销毁，进行清理操作

