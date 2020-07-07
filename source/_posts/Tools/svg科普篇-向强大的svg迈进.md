---
title: svg科普篇-向强大的svg迈进
tags: 'Svg'
categories: 'Tools'
date: 2020-07-01 09:29:27
---


# 前言

今天在掘金上突然看到京东凹凸实验室发布的一篇[《向强大的svg迈进》](https://juejin.im/post/5ef1a698f265da02ce2181d0?utm_source=gold_browser_extension)，想到前几个月项目老大非得让我从svg图标换成iconfont字体图标，最后在我耐心给老大讲了svg的未来，对比了iconfont与普通png的区别后，老大妥协了，让我说服其他成员，代码保持一致就行，:( ，没办法，又得给其他同事讲下。所以今天为了科普svg，写来这篇

> SVG 即 `Scalable Vector Graphics` 可缩放矢量图形，使用XML格式定义图形。
[原文地址](https://juejin.im/post/5ef1a698f265da02ce2181d0?utm_source=gold_browser_extension)
## SVG印象

SVG 的应用十分广泛，得益于 SVG 强大的各种特性。

### 矢量

可利用 SVG 矢量的特点，描出深圳地铁的轮廓：

![metro](https://user-gold-cdn.xitu.io/2020/6/23/172dff28ae2bb2f0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### iconfont

SVG 可依据一定的规则，转成 iconfont 使用：

![iconfont](https://user-gold-cdn.xitu.io/2020/6/23/172dff28e7152486?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### foreignObject

利用 SVG 的 `foreignObject` 标签实现截图功能，原理：`foreignObject` 内部嵌入 HTML 元素：

```js
<svg xmlns="http://www.w3.org/2000/svg">
	<foreignObject width="120" height="60">
		<p style="font-size:20px;margin:0;">凹凸实验室 欢迎您</p>
	</foreignObject>
</svg>
```

截图实现流程：

1.首先声明一个基础的 svg 模版，这个模版需要一些基础的描述信息，最重要的，它要有 `<foreignObject></foreignObject>` 这对标签；
2.将要渲染的 DOM 模版模版嵌入 `foreignObject` 即可；
3.利用 Blob 构建 svg 对象；
4.利用 `URL.createObjectURL(svg)` 取出 URL。

### SVG SMIL

由于微信编辑器不允许嵌入 `<style><script><a>` 标签，利用SVG SMIL 可进行微信公众号极具创意的图文排版设计，包括动画与交互。
但是也要注意，标签里不允许有id，否则会被过滤或替换掉。

点击 "凹凸实验室" 后，围绕 "凹凸实验室" 中心旋转 360度，点击0.5秒后 出现 [aotu.io/](https://aotu.io/) ，动画只运行一次。

下图为 GIF循环演示：

![demo](https://user-gold-cdn.xitu.io/2020/6/23/172dff2914fb651d?imageslim)

代码如下：

```js
<svg width="360" height="300" xmlns="http://www.w3.org/2000/svg">
    <g>
        <!-- 点击后 运行transform旋转动画，restart="never"表示只运行一次 -->
        <animateTransform attributeName="transform" type="rotate" begin="click" dur="0.5s" from="0 100 80" to="360 100 80"  fill="freeze" restart="never" />
        <g>
            <text font-family="microsoft yahei" font-size="20" x="50" y="80">
                凹凸实验室
            </text>
        </g>
        <g style="opacity: 0;">
            <!-- 同一个初始位置以及大致的宽高，触发点击事件 -->
            <text font-family="microsoft yahei" font-size="20" x="50" y="80">https://aotu.io/</text>
            <!-- 点击后 运行transform移动动画，改变文本的位置 -->
            <animateTransform attributeName="transform" type="translate" begin="click" dur="0.1s" to="0 40"  fill="freeze" restart="never" />
            <!-- 点击0.5秒后 运行opacity显示动画 -->
            <animate attributeName="opacity" begin="click+0.5s" from="0" to="1" dur="0.5s" fill="freeze" restart="never" />
        </g>
    </g>
</svg>
```

## SVG 实现非比例缩放

我们熟知的 `iconfont`，可通过改变字体大小缩放，但是这是 **比例缩放**，那如何实现 SVG 的**非比例缩放**呢？ 如下图所示，`如何将 一只兔子 非比例缩放？`

![demo](https://user-gold-cdn.xitu.io/2020/6/23/172dff2972d16d14?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

划重点：**实现非比例缩放主要涉及三个知识点：`viewport、viewBox和preserveAspectRatio`，`viewport 与viewBox` 结合可实现缩放的功能，`viewBox 与 preserveAspectRatio` 结合可实现非比例的功能**。

### viewport

`viewport` 表示SVG可见区域的大小。 `viewport` 就像是我们的显示器屏幕大小，超出区域则隐藏，原点位于左上角，x 轴水平向右，y 轴垂直向下。

![demo](https://user-gold-cdn.xitu.io/2020/6/23/172dff29abb23727?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

通过类似CSS的属性 `width、height` 指定视图大小：

```js
<svg width="400" height="200"></svg>
```

### viewBox

viewBox值有4个数字：`x, y, width, height` 。 其中 x：左上角横坐标，y：左上角纵坐标，width：宽度，height：高度。 
原点默认位于左上角，x 轴水平向右，y 轴垂直向下。

```js
<svg width="400" height="200" viewBox="0 0 200 100"></svg>
```

显示器屏幕的画面，可以特写，可以全景，这就是 `viewBox`。 `viewBox` 可以想象成截屏工具选中的那个框框，和 `viewport` 作用的结果就是 把框框中的截屏内容再次在 显示器 中全屏显示。

![demo](https://user-gold-cdn.xitu.io/2020/6/23/172dff29e1dee892?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### preserveAspectRatio

上图的红色框框和蓝色框框，恰好和显示器的比例相同，如果是下图的绿色框框，怎样在显示器屏幕中显示呢?

![demo](https://user-gold-cdn.xitu.io/2020/6/23/172dff2a1f0c94f7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 定义

`preserveAspectRatio` 作用的对象是 `viewBox`，使用方法如下：

```js
preserveAspectRatio="[defer] <align> [<meetOrSlice>]"
// 例如 preserveAspectRatio="xMidYMid meet"
```

其中 `defer` 此时不是重点，暂且忽略，主要了解 `align` 和 `meetOrSlice` 的 用法：

`align`：由两个名词组成，分别代表 `viewbox 与 viewport 的 x 方向、y方向`的对齐方式。

> `meetOrSlice`：表示如何维持高宽的比例，有三个值 `meet`、`slice`、`none`。
`meet` - 默认值，保持纵横比缩放 viewBox 适应 viewport，可能会有余留的空白。
`slice` - 保持纵横比同时比例小的方向放大填满 viewport，超出的部分被剪裁掉。
`none` - 扭曲纵横比以充分适应 viewport。

#### 例子

例子1：`preserveAspectRatio="xMidYMid meet"` 表示 绿色框框 与 显示器的 x 方向、y方向的 中心点 对齐；

![demo](https://user-gold-cdn.xitu.io/2020/6/23/172dff2a71a181c0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

例子2：`preserveAspectRatio="xMidYMin slice"` 表示 绿色框框 与 显示器的 x 方向 中心点 对齐，Y 方向 上边缘对齐，保持比例放大填满 显示屏 后超出部分隐藏；

![demo](https://user-gold-cdn.xitu.io/2020/6/23/172dff2aa4bade91?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

例子3：preserveAspectRatio="xMidYMid slice" 表示 绿色框框 与 显示器的 x 方向、y方向的 中心点 对齐，保持比例放大填满显示屏 后超出部分隐藏；

![demo](https://user-gold-cdn.xitu.io/2020/6/23/172dff2adcb56f90?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

例子4：preserveAspectRatio="none" 不管三七二十一，随意缩放绿色框框，填满 显示屏即可；这就是非比例缩放的答案了。

![demo](https://user-gold-cdn.xitu.io/2020/6/23/172dff2b10f609f4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## SVG vs Image, SVG vs Iconfont

SVG vs Image, SVG vs Iconfont 对比文章地址 [原文地址](https://blog.csdn.net/cpongo3/article/details/90258990?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-7.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-7.nonecase)