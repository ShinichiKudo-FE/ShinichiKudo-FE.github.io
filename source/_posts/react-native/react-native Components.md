---
title: react-native UI组件
date: 2018-02-11 16:13:39
tags: "react-native"
comments: true
categories: "react-native"
---
# React-native Components(自己理解)

[react-native 官网](http://facebook.github.io/react-native/)
[react-native 中文官网](https://reactnative.cn/docs/0.51/getting-started.html)

<!--more-->

**1.ActivityIndicator** 显示一个圆形的loading提示符号。
**2.Button** 按钮
**3.DataPickerIOS**  ios日期选择组件
**4.DrawerLayoutAndriod**   andriod抽屉导航
**5.FlatList**  高性能简单列表组件
**6.Image** 图片组件
**7.KeybordAvoiding View** 手机上弹出的键盘常常会挡住当前的视图。本组件可以自动根据键盘的位置，调整自身的position或底部的padding，以避免被遮挡
**8.ListView** 此组件已过期 请用selection 或 FlatLIst 替换
**9.MaskedViewIOS** 遮罩渲染子视图（添加炫酷的背景）
**10.Modal**  Modal组件可以用来覆盖包含React Native根视图的原生视图 (可用作弹框遮罩)
**11.NavigatorIOS**   IOS 导航设置
**12.Picker** 渲染原生的选择器（类似滚动地址选择）
**13.PickerIOS** IOS选择器
**14.ProgressBarAndroid**  安卓loading图标
**15.ProgressViewIOS**  IOS进度条
**16.RefreshControl**  这一组件可以用在ScrollView或ListView内部，为其添加下拉刷新的功能。当ScrollView处于竖直方向的起点位置（scrollY: 0），此时下拉会触发一个onRefresh事件。
**17.ScrollView** 滚动视图列表
**18.sectionList** 分组列表(例如电话本的列表)
**19.SegmentedControlIOS**  IOS分段选择器
**20.Slider** 用于选择一个范围值的组件。（例如选择音量的bar）
**21.SnapshotViewIOS**  IOS截图
**22.StatusBar**  用于控制应用状态栏的组件。
**23.Switch**  开关按钮
**24.TabBarIOS**  带IOS或Android后缀的组件(底部导航及)
**25.TabBarIOS.Item**    （底部导航的新消息通知数量）
**26.Text**  文本标签
**27.TextInput** 文本输入框
**28.ToolbarAndroid**  一个Toolbar可以显示一个徽标，一个导航图标（譬如汉堡形状的菜单按钮），一个标题与副标题，以及一个功能列表。
**29.TouchableHighlight**  本组件用于封装视图，使其可以正确响应触摸操作。当按下的时候，封装的视图的不透明度会降低，同时会有一个底层的颜色透过而被用户看到，使得视图变暗或变亮。
**30.TouchableWithoutFeedback** 响应用户的点击事件，如果你想在处理点击事件的同时不显示任何视觉反馈，使用它是个不错的选择。
**31. TouchableOpacity** 相比TouchableHighlight在按下去会使背景变暗的效果，TouchableOpacity会在用户手指按下时降低按钮的透明度，而不会改变背景的颜色
**32.TouchableNativeFeedback** 它会在用户手指按下时形成类似水波纹的视觉效果。注意，此组件只支持Android。
**33.View** 呈现的视图
**34.ViewPagerAndroid**   一个允许在子视图之间左右翻页的容器。每一个ViewPagerAndroid的子容器会被视作一个单独的页，并且会被拉伸填满ViewPagerAndroid。（注意所有的子视图都必须是纯View）
**35.VirtualizedList**    FlatList and SectionList 的底层实现。 通过维护一个有限的渲染窗口（其中包含可见的元素），并将渲染窗口之外的元素全部用合适的定长空白空间代替的方式，极大的改善了内存消耗以及在有大量数据情况下的使用性能。
**36.WebView**  用于包裹html的标签  （1.网址 2.引用路径 3.直接写html）
