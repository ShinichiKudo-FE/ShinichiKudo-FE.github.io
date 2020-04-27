---
title: React-native API
date: 2018-02-11 15:28:07
tags: "react-native"
comments: true
categories: "react-native"
---
# React-native API(自己理解)

最近在学习**react-native**,而且重新利用hexo 构建了自己的主页。 之前也在csdn,简书也写过博客，打算利用空闲时间把博客迁移进来。
进入主题吧。
        ------------------------------------------------
[react-native 官网](http://facebook.github.io/react-native/)
[react-native 中文官网](https://reactnative.cn/docs/0.51/getting-started.html)

<!--more-->

> * **1.AccessibilityInfo** 无障碍功能 给残障人士使用
> * **2.ActionSheetIOS**  IOS底部弹出选择框 （分享，选择）
> * **3.Alert** 调用手机原生弹框
> * **4.AlertIOS**  启动一个提示对话框，包含对应的标题和信息。
> * **5.Animated** Animated库使得开发者可以非常容易地实现各种各样的动画和交互方式，并且具备极高的性能
> * **6.AppRegistry**   AppRegistry是JS运行所有React Native应用的入口。
> * **7.AppState**  AppState能告诉你应用当前是在前台还是在后台，并且能在状态变化的时候通知你。
  AppState通常在处理推送通知的时候用来决定内容和对应的行为。
> * **8.AsyncStorage**  AsyncStorage是一个简单的、异步的、持久化的Key-Value存储系统，它对于App来说是全局性的。它用来代替LocalStorage。(react-native-storage模块，提供了较多便利功能。)
> * **9.BackHandler**   监听设备上的后退按钮事件。(仅限安卓)
> * **10.CameraRoll**  CameraRoll模块提供了访问本地相册的功能
> * **11.Clipboard**  Clipboard组件可以在iOS和Android的剪贴板中读写内容。
> * **12.DatePickerAndroid**  一个标准的Android日期选择器的对话框。
> * **13.Dimensions**  用于获取设备屏幕的宽高。
> * **14.Easing**   用于动画幅度
> * **15.Geolocation** 请求访问地理位置的权限
> * **16.ImageEditor**  根据指定的URI参数剪裁对应的图片
> * **17.ImagePickerIOS** 一个获取图片的 API（须百度详情）
> * **18.ImageStore**  图片缓存库
> * **19.Image Style Props**  可设置图片的属性样式
> * **20.InteractionManager** 提升用户体验和交互效果的模块InteractionMnager(交互管理器) 使用InteractionManager可以让一些耗时的任务在交互操作或者动画完成之后进行执行，这样使用可以保证我们的JavaScript的动画效果可以平滑流畅的执行。可以大大提升用户体验。（百度详情）
> * **21.Keyboard**  Keyboard组件可以用来控制键盘相关的事件。
> * **22.LayoutAnimation**  当布局变化时，自动将视图运动到它们新的位置上。
> * **23.Layout Props**  可以设置的动画样式参数
> * **24.Linking**  直接打开网址，电话，短信的链接。Linking模块给我们提供了Android和iOS双平台通用的接口进行处理App进入和传出的链接。(百度详情)
> * **25.ListViewDataSource**  为ListView组件提供高性能的数据处理和访问
> * **26.NativeMethodsMixin**  NativeMethodsMixin提供了一些用于直接访问底层原生组件的方法
> * **27.NetInfo**   模块可以获知设备联网或离线的状态信息。
> * **28.PanResponder**  可以将多点触摸操作协调成一个手势。它使得一个单点触摸可以接受更多的触摸操作，也可以用于识别简单的多点触摸手势。(百度详情)
> * **29.PermissionsAndroid**  Permissions权限适配
> * **30.PixelRatio**  根据像素密度获取指定大小的图片
> * **31.PushNotificationIOS**  本模块帮助你处理应用的推送通知，包括权限控制以及应用图标上的角标数（未读消息数）。（查看ApI或百度详情）
> * **32.Settings**  设置 （获取，设置，监听，清除key值）
> * **33.Shadow Props**  设置阴影样式参数
> * **34.Share** 打开一个对话框来共享文本内容。
> * **35.StatusBarIOS** 用于控制应用状态栏的组件。
> * **36.StyleSheet** 设置样式的属性
> * **37.Systrace**  内存优化
> * **38.Text Style Props**  文本样式参数
> * **39.TimePickerAndroid**  安卓日期选择
> * **40.ToastAndroid**  用于在Android设备上显示一个悬浮的提示信息
> * **41.Transforms**  动画样式参数
> * **42.Vibration** 用于控制设备震动
> * **43.VibrationIOS**  IOS版用于控制设备震动
> * **44.View Style Props**  设置视图样式参数从