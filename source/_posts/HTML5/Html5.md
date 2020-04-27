---
title: Html5
date: 2018-03-19 15:14:30
tags: "HTML5"
categories: "HTML5"
---
# html5特性

## 用于媒介回放的 video 和 audio 元素

### Video

#### 支持的视频格式

Ogg = 带有 Theora 视频编码和 Vorbis 音频编码的 Ogg 文件（.ogg）

MPEG4 = 带有 H.264 视频编码和 AAC 音频编码的 MPEG 4 文件(.mp4)

WebM = 带有 VP8 视频编码和 Vorbis 音频编码的 WebM 文件(.mkv)

```html
<video src="movie.ogg" controls="controls">
</video>
```

**controls** 属性供添加播放、暂停和音量控件。

包含宽度和高度属性也是不错的主意。
如果有不支持video的浏览器器则使用下面代码
```html
<video src="movie.ogg" width="320" height="240" controls="controls">
Your browser does not support the video tag.
</video>
```

video 元素允许多个 source 元素。source 元素可以链接不同的视频文件。浏览器将使用第一个可识别的格式：
```html
<video width="320" height="240" controls="controls">
  <source src="movie.ogg" type="video/ogg">
  <source src="movie.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>
```

IE8不支持video元素，在 IE 9 中，将提供对使用 MPEG4 的 video 元素的支持。

#### video标签属性：
autoplay 自动播放
controls 显示控件
width 设置播放器的宽度
height 设置播放器高度
loop  如果设置为True则循环播放
preload 如果出现该属性，则视频在页面加载时进行加载，并预备播放。如果使用 "autoplay"，则忽略该属性。
src 要播放的视频的 URL

#### video方法，属性及事件：
paly() 视频播放
pasue() 视频暂停
load()  视频加载

### audio

#### audio支持的音频格式
Ogg Vorbis （.ogg）
Mp3 (.mp3)
Wav (.wav)

#### audio标签的属性
autoplay 自动播放
controls 音频控件
loop 循环播放
preload 如果出现该属性，则视频在页面加载时进行加载，并预备播放。如果使用 "autoplay"，则忽略该属性。
src 要播放的音频的 URL

## html5拖放（drag和drop)
<!-- more -->
### 浏览器支持
Internet Explorer 9、Firefox、Opera 12、Chrome 以及 Safari 5 支持拖放。

注释：在 Safari 5.1.2 中不支持拖放。

一个简单地例子
```html
<!DOCTYPE HTML>
<html>
<head>
<script type="text/javascript">
function allowDrop(ev){
    ev.preventDefault();
}

function drag(ev){
    ev.dataTransfer.setData("Text",ev.target.id);
}

function drop(ev){
    ev.preventDefault();
    var data=ev.dataTransfer.getData("Text");
    ev.target.appendChild(document.getElementById(data));
}
</script>
</head>
<body>

<div id="div1" ondrop="drop(event)"
ondragover="allowDrop(event)"></div>
<img id="drag1" src="img_logo.gif" draggable="true"
ondragstart="drag(event)" width="336" height="69" />

</body>
</html>
```
### 设置元素为可拖放
首先，为了使元素可拖动，把 draggable 属性设置为 true ：
```html
<img draggable="true" />
```
### 拖动什么 - ondragstart 和 setData()
然后，规定当元素被拖动时，会发生什么。
在上面的例子中，ondragstart 属性调用了一个函数，drag(event)，它规定了被拖动的数据。
dataTransfer.setData() 方法设置被拖数据的数据类型和值：

```js
function drag(ev){
    ev.dataTransfer.setData("Text",ev.target.id);
}
```
在这个例子中，数据类型是 "Text"，值是可拖动元素的 id ("drag1")。

### 放到何处 - ondragover
ondragover 事件规定在何处放置被拖动的数据。

默认地，无法将数据/元素放置到其他元素中。如果需要设置允许放置，我们必须阻止对元素的默认处理方式。

这要通过调用 ondragover 事件的 event.preventDefault() 方法：
```js
event.preventDefault()
```

### 进行放置 - ondrop
当放置被拖数据时，会发生 drop 事件。

在上面的例子中，ondrop 属性调用了一个函数，drop(event)：

function drop(ev)
{
ev.preventDefault();
var data=ev.dataTransfer.getData("Text");
ev.target.appendChild(document.getElementById(data));
}

>代码解释：
1.调用 preventDefault() 来避免浏览器对数据的默认处理（drop 事件的默认行为是以链接形式打开）
2.通过 dataTransfer.getData("Text") 方法获得被拖的数据。该方法将返回在 setData() 方法中设置为相同类型的任何数据。
3.被拖数据是被拖元素的 id ("drag1")
4.把被拖元素追加到放置元素（目标元素）中

## Canvas(画布)
### 什么是 Canvas？
HTML5 的 canvas 元素使用 JavaScript 在网页上绘制图像。

画布是一个矩形区域，您可以控制其每一像素。
canvas 拥有多种绘制路径、矩形、圆形、字符以及添加图像的方法。

### 创建 Canvas 元素
向 HTML5 页面添加 canvas 元素。

规定元素的 id、宽度和高度：
```html
<canvas id="myCanvas" width="200" height="100"></canvas>
```
### 通过 JavaScript 来绘制
canvas 元素本身是没有绘图能力的。所有的绘制工作必须在 JavaScript 内部完成：
```js
<script type="text/javascript">
var c=document.getElementById("myCanvas");
var cxt=c.getContext("2d");
cxt.fillStyle="#FF0000";
cxt.fillRect(0,0,150,75);
</script>
```
JavaScript 使用 id 来寻找 canvas 元素：
```js
var c=document.getElementById("myCanvas");
```
然后，创建 context 对象：
```js
var cxt=c.getContext("2d"); 
getContext("2d") 对象是内建的 HTML5 对象，拥有多种绘制路径、矩形、圆形、字符以及添加图像的方法。
```
下面的两行代码绘制一个红色的矩形：
```js
cxt.fillStyle="#FF0000";
cxt.fillRect(0,0,150,75); 
```
fillStyle 方法将其染成红色，fillRect 方法规定了形状、位置和尺寸。

## SVG

### 什么是SVG？
SVG 指可伸缩矢量图形 (Scalable Vector Graphics)
SVG 用于定义用于网络的基于矢量的图形
SVG 使用 XML 格式定义图形
SVG 图像在放大或改变尺寸的情况下其图形质量不会有损失
SVG 是万维网联盟的标准

### SVG 的优势
与其他图像格式相比（比如 JPEG 和 GIF），使用 SVG 的优势在于：

SVG 图像可通过文本编辑器来创建和修改
SVG 图像可被搜索、索引、脚本化或压缩
SVG 是可伸缩的
SVG 图像可在任何的分辨率下被高质量地打印
SVG 可在图像质量不下降的情况下被放大

### 例子
```html
<!DOCTYPE html>
<html>
<body>

<svg xmlns="http://www.w3.org/2000/svg" version="1.1" height="190">
  <polygon points="100,10 40,180 190,60 10,60 160,180"
  style="fill:lime;stroke:purple;stroke-width:5;fill-rule:evenodd;" />
</svg>

</body>
</html>
```

## Canvas 与 SVG 的比较
下表列出了 canvas 与 SVG 之间的一些不同之处。

>Canvas
    依赖分辨率
    不支持事件处理器
    弱的文本渲染能力
    能够以 .png 或 .jpg 格式保存结果图像
    最适合图像密集型的游戏，其中的许多对象会被频繁重绘

>SVG
    不依赖分辨率
    支持事件处理器
    最适合带有大型渲染区域的应用程序（比如谷歌地图）
    复杂度高会减慢渲染速度（任何过度使用 DOM 的应用都不快）
    不适合游戏应用

## Html5地理定位
HTML5 Geolocation API 用于获得用户的地理位置。

鉴于该特性可能侵犯用户的隐私，除非用户同意，否则用户位置信息是不可用的。
```js
<script>
var x=document.getElementById("demo");
function getLocation()
  {
  if (navigator.geolocation)
    {
    navigator.geolocation.getCurrentPosition(showPosition);
    }
  else{x.innerHTML="Geolocation is not supported by this browser.";}
  }
function showPosition(position)
  {
  x.innerHTML="Latitude: " + position.coords.latitude +
  "<br />Longitude: " + position.coords.longitude;
  }
</script>
```

>上面代码解释：
检测是否支持地理定位
如果支持，则运行 getCurrentPosition() 方法。如果不支持，则向用户显示一段消息。
如果getCurrentPosition()运行成功，则向参数showPosition中规定的函数返回一个coordinates对象
showPosition() 函数获得并显示经度和纬度

处理错误：
```js
function showError(error)
  {
  switch(error.code)
    {
    case error.PERMISSION_DENIED:// 用户不允许地理定位
      x.innerHTML="User denied the request for Geolocation."
      break;
    case error.POSITION_UNAVAILABLE://无法获取当前位置
      x.innerHTML="Location information is unavailable."
      break;
    case error.TIMEOUT:// 超时处理
      x.innerHTML="The request to get user location timed out."
      break;
    case error.UNKNOWN_ERROR: // 错误处理
      x.innerHTML="An unknown error occurred."
      break;
    }
  }
```

## HTML 5 Web 存储
HTML5 提供了两种在客户端存储数据的新方法：

localStorage - 没有时间限制的数据存储
sessionStorage - 针对一个 session 的数据存储

### localStorage 方法
localStorage 方法存储的数据没有时间限制。第二天、第二周或下一年之后，数据依然可用。

如何创建和访问 localStorage：
```js
<script type="text/javascript">
localStorage.lastname="Smith";
document.write(localStorage.lastname);
</script>
```

### sessionStorage 方法
sessionStorage 方法针对一个 session 进行数据存储。当用户关闭浏览器窗口后，数据会被删除。

如何创建并访问一个 sessionStorage：
```js
<script type="text/javascript">
sessionStorage.lastname="Smith";
document.write(sessionStorage.lastname);
</script>
```

## HTML 5 应用程序缓存

### 什么是应用程序缓存（Application Cache）？
HTML5 引入了应用程序缓存，这意味着 web 应用可进行缓存，并可在没有因特网连接时进行访问。

>应用程序缓存为应用带来三个优势：

离线浏览 - 用户可在应用离线时使用它们
速度 - 已缓存资源加载得更快
减少服务器负载 - 浏览器将只从服务器下载更新过或更改过的资源。

### Cache Manifest 基础
如需启用应用程序缓存，请在文档的 <html> 标签中包含 manifest 属性：
```html
<!DOCTYPE HTML>
<html manifest="demo.appcache">
...
</html>
```

### Manifest 文件
manifest 文件是简单的文本文件，它告知浏览器被缓存的内容（以及不缓存的内容）。

manifest 文件可分为三个部分：

*CACHE MANIFEST* - 在此标题下列出的文件将在首次下载后进行缓存
*NETWORK* - 在此标题下列出的文件需要与服务器的连接，且不会被缓存
*FALLBACK* - 在此标题下列出的文件规定当页面无法访问时的回退页面（比如 404 页面）

### 更新缓存
一旦应用被缓存，它就会保持缓存直到发生下列情况：

用户清空浏览器缓存
manifest 文件被修改（参阅下面的提示）
由程序来更新应用缓存

### 实例 - 完整的 Manifest 文件
CACHE MANIFEST
 #2012-02-21 v1.0.0
/theme.css
/logo.gif
/main.js

NETWORK:
login.asp

FALLBACK:
/html5/ /404.html

<font color = "red">重要的提示：</font>以 "#" 开头的是注释行，但也可满足其他用途。应用的缓存会在其 manifest 文件更改时被更新。如果您编辑了一幅图片，或者修改了一个 JavaScript 函数，这些改变都不会被重新缓存。更新注释行中的日期和版本号是一种使浏览器重新缓存文件的办法。

注释：浏览器对缓存数据的容量限制可能不太一样（某些浏览器设置的限制是每个站点 5MB）。

## HTML 5 Web Workers
什么是 Web Worker？
当在 HTML 页面中执行脚本时，页面的状态是不可响应的，直到脚本已完成。

web worker 是运行在后台的 JavaScript，独立于其他脚本，不会影响页面的性能。您可以继续做任何愿意做的事情：点击、选取内容等等，而此时 web worker 在后台运行。


>Web Workers 和 DOM
由于 web worker 位于外部文件中，它们无法访问下例 JavaScript 对象：
window 对象
document 对象
parent 对象

## HTML 5 服务器发送事件
Server-Sent 事件 - 单向消息传递
Server-Sent 事件指的是网页自动获取来自服务器的更新。

以前也可能做到这一点，前提是网页不得不询问是否有可用的更新。通过服务器发送事件，更新能够自动到达。

例子：Facebook/Twitter 更新、估价更新、新的博文、赛事结果等
```js
var source=new EventSource("demo_sse.php");
source.onmessage=function(event)
  {
  document.getElementById("result").innerHTML+=event.data + "<br />";
  };
```
EventSource 对象
在上面的例子中，我们使用 onmessage 事件来获取消息。不过还可以使用其他事件：

onopen	当通往服务器的连接被打开
onmessage	当接收到消息
onerror	当错误发生

## html5表单
HTML5 新的 Input 类型
HTML5 拥有多个新的表单输入类型。这些新特性提供了更好的输入控制和验证。

本章全面介绍这些新的输入类型：

email 在提交表单时，会自动验证 email 域的值。
url 在提交表单时，会自动验证 url 域的值。
number number类型用于应该包含数值的输入域。
range range 类型显示为滑动条。
Date pickers (date, month, week, time, datetime, datetime-local) 日期选择器
search 搜索域，比如站点搜索或 Google 搜索。
color 

## HTML5 表单元素
>HTML5 的新的表单元素：
datalist datalist 元素规定输入域的选项列表。
```html
Webpage: <input type="url" list="url_list" name="link" />
<datalist id="url_list">
<option label="W3School" value="http://www.W3School.com.cn" />
<option label="Google" value="http://www.google.com" />
<option label="Microsoft" value="http://www.microsoft.com" />
</datalist>
```
提示：option 元素永远都要设置 value 属性。

keygen keygen 元素的作用是提供一种验证用户的可靠方法。
output output 元素用于不同类型的输出，比如计算或脚本输出：

## HTML5 表单属性

### 新的 form 属性：
autocomplete autocomplete 属性规定 form 或 input 域应该拥有自动完成功能。
novalidate

### 新的 input 属性：
autocomplete
autofocus autofocus 属性规定在页面加载时，域自动地获得焦点。
form
form overrides (formaction, formenctype, formmethod, formnovalidate, formtarget) 允许您重写 form 元素的某些属性设定
height 和 width height 和 width 属性规定用于 image 类型的 input 标签的图像高度和宽度。
list list 属性规定输入域的 datalist。datalist 是输入域的选项列表。
min, max 和 step step 属性为输入域规定合法的数字间隔
multiple multiple 属性适用于以下类型的 <input> 标签：email 和 file。
pattern (regexp) pattern 属性规定用于验证 input 域的模式（pattern）。
placeholder 属性提供一种提示（hint），描述输入域所期待的值。
required  属性规定必须在提交之前填写输入域（不能为空）。