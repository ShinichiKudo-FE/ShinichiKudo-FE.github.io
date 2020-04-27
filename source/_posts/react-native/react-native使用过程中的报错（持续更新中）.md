---
title: react-native使用过程中的报错（持续更新中）
date: 2018-03-01 17:36:11
tags: "react-native"
categories: "react-native"
---
# ReactNative 遇到的报错

## react-native 矢量库 react-native-vector-icons的报错解决

![imgerror](http://img.blog.csdn.net/20180206155016745?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzc0ODEwODE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
解决方法：将项目目录下 node_modules/react-native/local-cli/core/__fixtures__/files/package.json删除
<!-- more -->
如果还出现这个错误
![imgerror1](http://img.blog.csdn.net/20180206155244544?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzc0ODEwODE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

解决方案：删除重新安装react-native-vector-icons 安装完成后执行
 react-native link 继续修改问题1的错误

## unRecognized font family 'XXXXX'
解决方案：
![imgerror2](http://img.blog.csdn.net/20180206155743569?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzc0ODEwODE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## react-native开发踩坑之 ios上react-native-vector-icons 的error：unRecognized font family 'FontAwesome'
![imgerror3](http://img.blog.csdn.net/20170405222455738?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamlhbmdjczUyMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
接下来解释一下，首先看第一步：把node_modules/react-native-vector-icons下的fonts文件添加到工程中，这时候往往忽略了一个重点，后面那一句

```javascript
Make sure your app is checked under “Add to targets” and that “Create groups” is checked if you add the whole folder
```
这一句很关键，意思是确认必须要确认是Add to targets和Create groups，一开始我是直接右键把fonts目录add到工程的，根本没看到这两个玩意，结果就是失败，有点奇怪，后来我仔细看了一下这个步骤描述，

```javascript
drag the folder Fonts to your project in Xcode
```

## ReactNative Ios报出 'React/RCTBundleURLProvider.h' file not found错误
我在创建react-native项目时  npm了一个第三方库  结果一打开 xcode 竟然报错 React/RCTBundleURLProvider.h' file not found；

然后 我试了各种网上的方法试了一遍， 还是不行

就在我要放弃的时候 ，突然想到把网上的方法 结合一下 试试，结果好了。

以下就是我的解决方法
> 1.打开Mac里面的终端，进入项目所在的文件夹目录；
2.把项目里面的 node_modules 文件夹删除掉，然后执行 npm install 命令
3.npm install安装完成后， 执行react-native upgrade命令
4.npm start
5.打开xcode (如果还报错)
6.左侧点击跟项目目录 -> 选择右侧 Build Settings -> 选择 All & Combined -> 搜索框输入 Always Search User Paths -> 将 Always Search User Paths 设置为 Yes -> Clean -> Build
![imgerror4](https://img.hacpai.com/file/2017/6/5951e4a8885d4c37a63a0a275f893c99-image.png)

## RN SectionList 遇到 missing keys for items, make sure to specify a key property on each item 的 问题解决
![imgerror5](http://img.blog.csdn.net/20180107172034312?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzc0ODEwODE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
它警告我们每个item 要有不同的key

默认情况下每行都需要提供一个不重复的key属性。你也可以提供一个keyExtractor函数来生成key。
把这个属性添加到     
 <SectionList/> 里面
 ```javascript
       keyExtractor = {this._extraUniqueKey}   
         _extraUniqueKey(item ,index){
        eturn "index"+index+item;
        }     
```

这是每个item要设置key, 同样每个子控件也不能放过， 一定要设置它的key， 要不然这个屎黄色一直伴着你 多烦~~

## react-native 运行时，出现如下报错：
![imgerror6](https://upload-images.jianshu.io/upload_images/5227057-07eba5a0ae5b9f0c.png)
首先试试react-native start 或 npm start 看看是否报错，如果报错出现ERROR:Metro Bundler can't listen on port 8081

则是8081端口号被占用

解决方案：执行sudo lsof -i :8081  kill -9 进程号

![imgerrot7](https://upload-images.jianshu.io/upload_images/5227057-e66a166a6e956daa.png)

## 更多ReactNative问题总结集合，请移步大佬链接
[react-native 问题](https://www.jianshu.com/p/98c8f2a970eb)

