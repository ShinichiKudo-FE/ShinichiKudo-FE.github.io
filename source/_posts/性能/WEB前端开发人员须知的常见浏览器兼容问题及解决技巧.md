---
title: WEB前端开发人员须知的常见浏览器兼容问题及解决技巧
date: 2018-04-23 15:30:44
tags: "浏览器兼容性"
categories: "兼容性"
---

## 为什么会有兼容问题？

由于市场上浏览器种类众多，而不同浏览器其内核亦不尽相同，所以各个浏览器对网页的解析就有一定出入，这也是导致浏览器兼容问题出现的主要原因，我们的网页需要在主流浏览器上正常运行，就需要做好浏览器兼容。

使用Trident内核的浏览器：IE、Maxthon、TT； 使用Gecko内核的浏览器：Netcape6及以上版本、FireFox； 使用Presto内核的浏览器：Opera7及以上版本； 使用Webkit内核的浏览器：Safari、Chrome。

而我现在所说的兼容性问题，主要是说IE与几个主流浏览器如firefox，google等。而对IE浏览器来说，IE7又是个跨度，因为之前的版本更新甚慢，bug甚多。从IE8开始，IE浏览器渐渐遵循标准，到IE9后由于大家都一致认为标准很重要，可以说在兼容性上比较好了，但是在中国来说，由于xp的占有率问题，使用IE7以下的用户仍然很多，所以我们不得不考虑低版本浏览器的兼容。

对浏览器兼容问题，一般分，HTML，Javascript兼容，CSS兼容。 其中html相关问题比较容易处理，无非是高版本浏览器用了低版本浏览器无法识别的元素，导致其不能解析，所以平时注意一点就是。特别是HTML5增加了许多新标签，低版本浏览器有点影响时代进步啊

### 问题一：不同浏览器的标签默认的外边距和内边距不同

问题症状：随便写几个标签，不加样式控制的情况下，各自的margin 和padding差异较大。
碰到频率:100%
解决方案：css里 *{margin:0;padding:0;}
备注：这个是最常见的也是最易解决的一个浏览器兼容性问题，几乎所有的css文件开头都会用通配符*来设置各个标签的内外补丁是0。

### 问题二：块属性标签float后，又有横行的margin情况下，在ie6显示margin比设置的大

问题症状:常见症状是ie6中后面的一块被顶到下一行
碰到频率：90%（稍微复杂点的页面都会碰到，float布局最常见的浏览器兼容问题）
解决方案：在float的标签样式控制中加入 display:inline;将其转化为行内属性
备注：我们最常用的就是div+css布局了，而div就是一个典型的块属性标签，横向布局的时候我们通常都是用div float实现的，横向的间距设置如果用margin实现，这就是一个必然会碰到的兼容性问题。

### 问题三：设置较小高度标签（一般小于10px），在ie6，ie7，遨游中高度超出自己设置高度

问题症状：ie6、7和遨游里这个标签的高度不受控制，超出自己设置的高度
碰到频率：60%
解决方案：给超出高度的标签设置overflow:hidden;或者设置行高line-height 小于你设置的高度。
备注：这种情况一般出现在我们设置小圆角背景的标签里。出现这个问题的原因是ie8之前的浏览器都会给标签一个最小默认的行高的高度。即使你的标签是空的，这个标签的高度还是会达到默认的行高。

<!-- more -->

### 问题四：行内属性标签，设置display:block后采用float布局，又有横行的margin的情况，ie6间距bug（类似第二种）

问题症状：ie6里的间距比超过设置的间距
碰到几率：20%
解决方案：在display:block;后面加入display:inline;display:table;
备注：行内属性标签，为了设置宽高，我们需要设置display:block;(除了input标签比较特殊)。在用float布局并有横向的margin后，在ie6下，他就具有了块属性float后的横向margin的bug。不过因为它本身就是行内属性标签，所以我们再加上display:inline的话，它的高宽就不可设了。这时候我们还需要在display:inline后面加入display:talbe。

### 图片默认有间距

问题症状：几个img标签放在一起的时候，有些浏览器会有默认的间距，加上问题一中提到的通配符也不起作用。
碰到几率：20%
解决方案：使用float属性为img布局
备注：因为img标签是行内属性标签，所以只要不超出容器宽度，img标签都会排在一行里，但是部分浏览器的img标签之间会有个间距。去掉这个间距使用float是正道

### 标签最低高度设置min-height不兼容

问题症状：因为min-height本身就是一个不兼容的css属性，所以设置min-height时不能很好的被各个浏览器兼容
碰到几率：5%
解决方案：如果我们要设置一个标签的最小高度200px，需要进行的设置为：{min-height:200px; height:auto !important; height:200px; overflow:visible;}
备注：在B/S系统前端开时，有很多情况下我们有这种需求。当内容小于一个值（如300px）时。容器的高度为300px；当内容高度大于这个值时，容器高度被撑高，而不是出现滚动条。这时候我们就会面临这个兼容性问题。

### 问题七：透明度的兼容css设置

方法是：每写一小段代码（布局中的一行或者一块）我们都要在不同的浏览器中看是否兼容，当然熟练到一定的程度就没这么麻烦了。建议经常会碰到兼容性问题的新手使用。很多兼容性问题都是因为浏览器对标签的默认属性解析不同造成的，只要我们稍加设置都能轻松地解决这些兼容问题。如果我们熟悉标签的默认属性的话，就能很好的理解为什么会出现兼容问题以及怎么去解决这些兼容问题。

### 技巧一：css hack

使用hacker 我可以把浏览器分为3类：`ie6` ；`ie7和遨游`；`其他（ie8 chrome ff safari opera等）`

ie6认识的hacker 是下划线_ 和星号 *
ie7 遨游认识的hacker是星号 * （包括上面问题6中的 !important也算是hack的一种。不过实用性较小。）

因为优先级相同且相冲突的属性设置后一个会覆盖掉前一个，所以书写的次序是很重要的。

越少的浮动，就会越少的代码，会有更灵活的页面，会有扩展性更强的页面。这不多说，归结为到一定水平了，浮动会用的较少。另外，您也会避免使用浮动+margin的用法。所以，越后来越不易遇到这种bug。

### 技巧二：padding，marign，height，width

注意是技巧，不是方法： 写好标准头 http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd”> http://www.w3.org/1999/xhtml”> **尽量用padding，慎用margin**，height尽量补上100%，父级height有定值子级height不用100%，子级全为浮动时底部补个空clear:both的div宽尽量用margin，慎用padding，width算准实际要的减去padding

### 技巧三：显示类（display:block,inline）

display:block,inline两个元素

display:block; //可以为内嵌元素模拟为块元素

display:inline; //实现同一行排列的的效果

display:table; //for FF,模拟table的效果

### 技巧四：怎样使一个div层居中于浏览器中？

```html
<style type="text/css">

div {

position:absolute;
top:50%;
left:50%;
margin:-100px 0 0 -100px;
width:200px;
height:200px;
border:1px solid red; }

</style>
```

2）div里的内容，IE默认为居中，而FF默认为左对齐，可以尝试增加代码margin: 0 auto;

### 技巧五：float的div闭合;清除浮动;自适应高度

① 例如：＜div id="floatA">＜div id="floatB">＜div id="NOTfloatC">

②作为外部 wrapper 的 div 不要定死高度,为了让高度能自适应，要在wrapper里面加上overflow:hidden; 当包含float的box的时候，高度自适应在IE下无效，这时候应该触发IE的layout私有属性(万恶的IE啊！)用zoom:1;可以做到，这样就达到了兼容。 

③对于排版,我们用得最多的css描述可能就是float:left.有的时候我们需要在n栏的float div后面做一个统一的背景,

④万能float 闭合(非常重要!)

关于 clear float 的原理可参见 [How To ClearFloats Without Structural Markup],将以下代码加入Global CSS 中,给需要闭合的div加上class=”clearfix”即可,屡试不爽。

### html5shiv.js

解决 ie9 以下浏览器对 html5 新增标签不识别的问题。

```javascript
<!--[if lt IE 9]>
  <script type="text/javascript" src="https://cdn.bootcss.com/html5shiv/3.7.3/html5shiv.min.js"></script>
<![endif]-->
```

### BFC 解决边距重叠问题

当相邻元素都设置了 margin 边距时，margin 将取最大值，舍弃小值。为了不让边距重叠，可以给子元素加一个父元素，并设置该父元素为 BFC：overflow: hidden;

### IE6 双倍边距的问题

设置 ie6 中设置浮动，同时又设置 margin，会出现双倍边距的问题
display: inline;

### 解决 IE9 以下浏览器不能使用 opacity

opacity: 0.5;
filter: alpha(opacity = 50);
filter: progid:DXImageTransform.Microsoft.Alpha(style = 0, opacity = 50);

### 解决 IE6 不支持 fixed 绝对定位以及IE6下被绝对定位的元素在滚动的时候会闪动的问题

```css
/* IE6 hack */
*html, *html body {
  background-image: url(about:blank);
  background-attachment: fixed;
}
*html #menu {
  position: absolute;
  top: expression(((e=document.documentElement.scrollTop) ? e : document.body.scrollTop) + 100 + 'px');
}
```

### IE6 背景闪烁的问题

问题：链接、按钮用 CSSsprites 作为背景，在 ie6 下会有背景图闪烁的现象。原因是 IE6 没有将背景图缓存，每次触发 hover 的时候都会重新加载

解决：可以用 JavaScript 设置 ie6 缓存这些图片：

```css
document.execCommand("BackgroundImageCache", false, true);
```

### 解决在 IE6 下，列表与日期错位的问题

日期`span` 标签放在标题 `a` 标签之前即可

### 解决 IE6 不支持 min-height 属性的问题

min-height: 350px;
_height: 350px;

### 让 IE7 IE8 支持 CSS3 background-size属性

```css
html {
  height: 100%;
}
body {
  height: 100%;
  margin: 0;
  padding: 0;
  background-image: url('img/37.png');
  background-repeat: no-repeat;
  background-size: cover;
  -ms-behavior: url('css/backgroundsize.min.htc');
  behavior: url('css/backgroundsize.min.htc');
}
```

### IE6-7 line-height 失效的问题

问题：在ie 中 img 与文字放一起时，line-height 不起作用

解决：都设置成 float

### td 自动换行的问题

问题：table 宽度固定，td 自动换行

解决：设置 Table 为 table-layout: fixed，td 为 word-wrap: break-word

### 键盘事件 keyCode 兼容性写法

```js
var inp = document.getElementById('inp')
var result = document.getElementById('result')

function getKeyCode(e) {
  e = e ? e : (window.event ? window.event : "")
  return e.keyCode ? e.keyCode : e.which
}

inp.onkeypress = function(e) {
  result.innerHTML = getKeyCode(e)
}
```

### 求窗口大小的兼容写法

```js
// 浏览器窗口可视区域大小（不包括工具栏和滚动条等边线）
// 1600 * 525
var client_w = document.documentElement.clientWidth || document.body.clientWidth;
var client_h = document.documentElement.clientHeight || document.body.clientHeight;

// 网页内容实际宽高（包括工具栏和滚动条等边线）
// 1600 * 8
var scroll_w = document.documentElement.scrollWidth || document.body.scrollWidth;
var scroll_h = document.documentElement.scrollHeight || document.body.scrollHeight;

// 网页内容实际宽高 (不包括工具栏和滚动条等边线）
// 1600 * 8
var offset_w = document.documentElement.offsetWidth || document.body.offsetWidth;
var offset_h = document.documentElement.offsetHeight || document.body.offsetHeight;

// 滚动的高度
var scroll_Top = document.documentElement.scrollTop||document.body.scrollTop;

```