---
title: '强大的CSS:文字下波浪线动画效果'
date: 2020-04-23 13:01:46
tags: "CSS"
categories: "CSS"
---

[原文地址](https://www.imooc.com/article/286988)

![wave](http://upload-images.jianshu.io/upload_images/13133049-963e2b89fb9da2ad.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

类似下面这种效果
![text wave](http://upload-images.jianshu.io/upload_images/13133049-03fa0c4c990af8f1.gif?imageMogr2/auto-orient/strip)

有以下几种方式可以实现这种效果

## 径向渐变纯CSS的方式

就是使用径向渐变绘制我们的波浪线效果，一个波浪线循环段是有一个朝上的半个圆弧和一个朝下的半个圆弧组合而成的。

所以，我们只要使用径向渐变绘制圆弧，再通过`background-position`控制两个圆弧的位置，让其前后拼接在一起就可以实现波浪线效果。

相关CSS代码

```css
.flow-wave {
    background: radial-gradient(circle at 10px -7px, transparent 8px, currentColor 8px, currentColor 9px, transparent 9px) repeat-x,
        radial-gradient(circle at 10px 27px, transparent 8px, currentColor 8px, currentColor 9px, transparent 9px) repeat-x;
    background-size: 20px 20px;
    background-position: -10px calc(100% + 16px), 0 calc(100% - 4px);
}
```

实时效果如下：
有了静态的波浪线效果，剩下的就像这个波浪线动起来，用`CSS3 animation`动画控制`background-position`位置即可。

```css
.flow-wave {
    animation: waveFlow 1s infinite linear;
}
@keyframes waveFlow {
    from { background-position-x: -10px, 0; }
    to { background-position-x: -30px, -20px; }
}
```

于是波浪线动画效果就实现了。

这种方法实现的优点是颜色控制非常方便，例如，我们修改文字颜色为原谅绿，弯弯水波的颜色也变成了原谅绿：

此方法实现的不足就是：理解成本有点高，如何使用CSS radial-gradient模拟波浪线效果是需要相当的CSS功力积累的，上手不太容易，以后要修改个尺寸什么的也不太好维护。同时，在普通屏幕密度屏幕下的Chrome浏览器上，线条并不平滑，吹毛求疵的设计师不一定会接受。

此时，可以试试下面这种更利于理解的方法。

## 使用SVG波形矢量图作为背景

就是我们直接使用一个使用SVG波形矢量图作为背景，不用自己去手动CSS绘制，代码量差不多，还更容易理解。CSS代码示意：

```css
svg-wave {
    background: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 20 4'%3E%3Cpath fill='none' stroke='%23333' d='M0 3.5c5 0 5-3 10-3s5 3 10 3 5-3 10-3 5 3 10 3'/%3E%3C/svg%3E") repeat-x 0 100%; 
    background-size: 20px auto;
}
```

实时效果如下：

有了静态的波浪线效果，剩下的就像这个波浪线动起来，用`CSS3 animation`动画控制`background-position`位置即可。

```css 
.svg-wave {
    animation: waveMove 1s infinite linear;
}
@keyframes waveMove {
    from { background-position: 0 100%; }
    to   { background-position: -20px 100%; }
}
```

优点是线条边缘平滑，效果细腻，易理解，易上手，易维护。

缺点也很明显，就是波浪线的颜色无法实时跟着文字的颜色发生变化，适用于文字颜色不会多变的场景。

如果我们想要改变波浪线的颜色也很简单，修改`background`代码中的`stroke='%23333'`这部分，'%23'就是就是#，因此，`stroke='%23333'`其实就是`stroke='#333'`的意思。例如，我们需要改成红色略带橙色，可以`stroke='%23F30'`，也可以写完整`stroke='%23FF3300'`。


## text-decoration

`text-decoration`属性早已支持波浪线下划线：

```css
text-decoration-style: wavy;
```

效果如下截图：

![text-decoration-style](http://upload-images.jianshu.io/upload_images/13133049-0b1b797b8f9d40aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到，线好粗好不协调，而且字符和装饰线发生重叠的时候，装饰线直接消失了，结果波浪线变成了一截一截的，还需要使用text-decoration-skip进行额外控制。因此，实际开发，不建议在实际项目中使用。

而且你无法预知每个波浪线重复片段的宽度，想要无限运动理论上就不太可行。

因此，文字或者图形的波浪线动画效果不能使用text-decoration的波浪线。


