title: Android中进程的优先级
date: 2016/3/5 18:27:44       
categories: Android
---

# Android中进程的优先级 #

1. 前台进程：拥有一个正在与用户交互的Activity（onResume方法被调用）的进程
2. 可见进程：拥有一个可见但是没有焦点的Activity（onPause方法被调用）
3. 服务进程：拥有一个通过startService方法启动的服务
4. 后台进程：拥有一个不可见的Activity（onStop方法被调用）的进程
5. 空进程：没有拥有任何活动的应用组件的进程