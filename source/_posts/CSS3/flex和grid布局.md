---
title: flex和grid布局
date: 2018-03-06 15:56:34
tags: "Css3"
categories: "Css3"
---
# 网页布局（layout）是 CSS 的一个重点应用。

![layout](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071001.gif)

布局的传统解决方案，基于盒状模型，依赖 display 属性 + position属性 + float属性。它对于那些特殊布局非常不方便，比如，垂直居中就不容易实现。


## Flex 布局

2009年，W3C 提出了一种新的方案----Flex 布局，可以简便、完整、响应式地实现各种页面布局。目前，它已经得到了所有浏览器的支持，这意味着，现在就能很安全地使用这项功能。

[原文地址](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)
[Demo地址](http://static.vgee.cn/static/index.html)

### 容器属性

#### flex-direction属性

flex-direction属性决定主轴的方向（即项目的排列方向）。
```javascript
.box{
    flex-direction: row | row-reverse | column | column-reverse
}
```

>row（默认值）：主轴为水平方向，起点在左端。
 row-reverse：主轴为水平方向，起点在右端。
 column：主轴为垂直方向，起点在上沿。
 column-reverse：主轴为垂直方向，起点在下沿。

![flex1](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071005.png)

#### flex-wrap属性
默认情况下，项目都排在一条线（又称"轴线"）上。flex-wrap属性定义，如果一条轴线排不下，如何换行。

```javascript
.box{
  flex-wrap: nowrap | wrap | wrap-reverse;
}
```

<!-- more -->
![flex2](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071006.png)

>nowrap（默认）：不换行。
wrap：换行，第一行在上方。
wrap-reverse：换行，第一行在下方。

#### flex-flow属性
<font color = "red">flex-flow属性</font>是flex-direction属性和flex-wrap属性的简写形式，默认值为row nowrap。

```javascript
.box {
  flex-flow: <flex-direction> || <flex-wrap>;
}
```

#### justify-content属性
<font color = "red">justify-content属性</font>定义了项目在主轴上的对齐方式。

```javascript
.box {
  justify-content: flex-start | flex-end | center | space-between | space-around;
}
```

>flex-start（默认值）：左对齐
flex-end：右对齐
center： 居中
space-between：两端对齐，项目之间的间隔都相等。
space-around：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。

#### align-items属性
<font color = "red">align-items属性</font>定义项目在交叉轴上如何对齐

```javascript
.box {
  align-items: flex-start | flex-end | center | baseline | stretch;
}
```

>flex-start：交叉轴的起点对齐。
flex-end：交叉轴的终点对齐。
center：交叉轴的中点对齐。
baseline: 项目的第一行文字的基线对齐。
stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。

#### align-centet属性
<font color = "red">align-content属性</font>定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。

```javascript
.box {
  align-content: flex-start | flex-end | center | space-between | space-around | stretch;
}
```

>flex-start：与交叉轴的起点对齐。
flex-end：与交叉轴的终点对齐。
center：与交叉轴的中点对齐。
space-between：与交叉轴两端对齐，轴线之间的间隔平均分布。
space-around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
stretch（默认值）：轴线占满整个交叉轴。

### 项目属性
以下设置在项目上。

#### order属性
<font color = "red">order属性</font>定义项目的排列顺序。数值越小，排列越靠前，默认为0。

```javascript
.item {
  order: <integer>;
}
```

#### flex-grow属性
<font color = "red">flex-grow属性</font>定义项目的放大比例，默认为0，即如果存在剩余空间，也不放大。

```javascript
.item {
  flex-grow: <number>; /* default 0 */
}
```
如果所有项目的**flex-grow**属性都为1，则它们将等分剩余空间（如果有的话）。如果一个项目的**flex-grow**属性为2，其他项目都为1，则前者占据的剩余空间将比其他项多一倍。

#### flex-shrink属性
<font color = "red">flex-shrink属性</font>定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。

```javascript
.item {
  flex-shrink: <number>; /* default 1 */
}
```

如果所有项目的**flex-shrink**属性都为1，当空间不足时，都将等比例缩小。如果一个项目的flex-shrink属性为0，其他项目都为1，则空间不足时，前者不缩小。

<font color = "red">负值对该属性无效</font>。

#### flex-basis属性
<font color = "red">flex-basis属性</font>定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为auto，即项目的本来大小。

```javascript
.item {
  flex-basis: <length> | auto; /* default auto */
}
```

它可以设为跟*width*或*height*属性一样的值（比如350px），则项目将占据固定空间。

#### flex属性
flex属性是*flex-grow,* *flex-shrink* 和 *flex-basis*的简写，默认值为0 1 auto。后两个属性可选。

```javascript
.item {
  flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
}
```


该属性有两个快捷值：**auto (1 1 auto)** 和 **none (0 0 auto)**。

建议优先使用这个属性，而不是单独写三个分离的属性，因为浏览器会推算相关值。

#### align-self属性
<font color = "red">align-self属性</font>允许单个项目有与其他项目不一样的对齐方式，可覆盖<font color = "red">align-items属性</font>。默认值为<font color = "red">auto</font>，表示继承父元素的<font color = "red">align-items属性</font>，如果没有父元素，则等同于stretch

```javascript
.item {
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```
该属性可能取6个值，除了auto，其他都与align-items属性完全一致。


## Grid布局
[原文地址](http://www.css88.com/archives/8510)

Grid 布局是网站设计的基础，CSS Grid 是创建网格布局最强大和最简单的工具。

CSS Grid 今年也获得了主流浏览器（Safari，Chrome，Firefox，Edge）的原生支持，所以我相信所有的前端开发人员都必须在不久的将来学习这项技术。

CSS Grid(网格) 布局（又称为 “Grid(网格)” ），是一个二维的基于网格的布局系统它的目标是完全改变我们基于网格的用户界面的布局方式。CSS 一直用来布局我们的网页，但一直以来都存在这样或那样的问题。一开始我们用表格（table），然后是浮动（float），再是定位（postion）和内嵌块（inline-block），但是所有这些方法本质上都是只是 hack 而已，并且遗漏了很多重要的功能（例如垂直居中）。Flexbox 的出现很大程度上改善了我们的布局方式，但它的目的是为了解决更简单的一维布局，而不是复杂的二维布局（实际上 Flexbox 和 Grid 能结合在一起工作，而且配合得非常好）。Grid(网格) 布局是第一个专门为解决布局问题而创建的 CSS 模块，我们终于不需要想尽办法hack 页面布局样式了。

![grid](http://newimg88.b0.upaiyun.com/newimg88/2017/12/1_Oc88rInEcNuY-xCN3e1iPQ.png)

### 网格容器(Grid Container)

应用 display: grid 的元素。这是所有网格项（Grid Items）的直接父级元素。在这个例子中，container 就是 网格容器(Grid Container)。

```html
<div class="container">
  <div class="item item-1"></div>
  <div class="item item-2"></div>
  <div class="item item-3"></div>
</div>
```
#### display
>grid ：生成一个块级网格
inline-grid ：生成一个内联网格
subgrid ：如果你的网格容器本身是另一个网格的网格项（即嵌套网格），你可以使用这个属性来表示
  你希望它的行/列的大小继承自它的父级网格容器，而不是自己指定的。

如果你想要设置为网格容器元素本身已经是网格项（嵌套网格布局），用这个属性指明这个容器内部的网格项的行列尺寸直接继承其父级的网格容器属性。

```css
.container {
    display: grid | inline-grid | subgrid;
}
```
注意：在*网格容器(Grid Container) 上使用column，float，clear， vertical-align*不会产生任何效果。

#### grid-template-columns / grid-template-rows
使用空格分隔的值列表，用来定义网格的列和行。这些值表示 网格轨道(Grid Track) 大小，它们之间的空格表示网格线。
>值：
– <track-size>： 可以是长度值，百分比，或者等份网格容器中可用空间（使用 fr 单位）
– <line-name>：你可以选择的任意名称

```css
.container {
    grid-template-columns: <track-size> ... | <line-name> <track-size> ...;
    grid-template-rows: <track-size> ... | <line-name> <track-size> ...;
}
```

**示例：**
![grid3](http://newimg88.b0.upaiyun.com/newimg88/2017/12/grid-numbers.png)
```css
.container{
    grid-template-columns: 40px 50px auto 50px 40px;
    grid-template-rows: 25% 100px auto;
}
```

但是你可以明确的指定网格线(Grid Line)名称，即 <line-name> 值。请注意网格线名称的括号语法：

```css
.container {
    grid-template-columns: [first] 40px [line2] 50px [line3] auto [col4-start] 50px [five] 40px [end];
    grid-template-rows: [row1-start] 25% [row1-end] 100px [third-line] auto [last-line];
}
```

请注意，一条网格线(Grid Line)可以有多个名称。例如，这里的第二条 行网格线(row grid lines) 将有两个名字：row1-end 和row2-start ：
```css
.container{
    grid-template-rows: [row1-start] 25% [row1-end row2-start] 25% [row2-end];
}
```

如果你的定义包含多个重复值，则可以使用 repeat() 表示法来简化定义：
```css
.container {
    grid-template-columns: repeat(3, 20px [col-start]) 5%;
}
//等价于
.container {
    grid-template-columns: 20px [col-start] 20px [col-start] 20px [col-start] 5%;
}
```
fr 单元允许你用等分网格容器剩余可用空间来设置 网格轨道(Grid Track) 的大小 。例如，下面的代码会将每个网格项设置为网格容器宽度的三分之一：
```css
.container {
    grid-template-columns: 1fr 1fr 1fr;
}
```

剩余可用空间是除去所有非灵活网格项之后计算得到的。在这个例子中，可用空间总量减去 50px 后，再给 fr 单元的值3等分：
```css
.container {
    grid-template-columns: 1fr 50px 1fr 1fr;
}
```
#### grid-template-areas
通过引用 grid-area 属性指定的 网格区域(Grid Area) 名称来定义网格模板。重复网格区域的名称导致内容跨越这些单元格。一个点号（.）代表一个空的网格单元。这个语法本身可视作网格的可视化结构。

><grid-area-name>：由网格项的 grid-area 指定的网格区域名称
.（点号） ：代表一个空的网格单元
none：不定义网格区域

```css
.container {
  grid-template-areas: 
    " | . | none | ..."
    "...";
}
```

**示例：**
```css
.item-a {
  grid-area: header;
}
.item-b {
  grid-area: main;
}
.item-c {
  grid-area: sidebar;
}
.item-d {
  grid-area: footer;
}
 
.container {
  grid-template-columns: 50px 50px 50px 50px;
  grid-template-rows: auto;
  grid-template-areas: 
    "header header header header"
    "main main . sidebar"
    "footer footer footer footer";
}
```
上面的代码将创建一个 4 列 3 行的网格。整个顶行将由 header 区域 组成。中间一排将由两个 main 区域，一个是空单元格，一个 sidebar 区域组成。最后一行全是 footer 区域组成。
![grid4](http://newimg88.b0.upaiyun.com/newimg88/2017/12/grid-template-areas.png)

你可以使用任意数量的相邻的 <font color="red">点.</font> 来声明单个空单元格。 只要这些点.之间没有空隙隔开，他们就表示一个单一的单元格。

#### grid-template
用于定义 **grid-template-rows**，**grid-template-columns** ，**grid-template-areas** 缩写 (shorthand) 属性。

>none：将所有三个属性设置为其初始值
subgrid：将grid-template-rows，grid-template-columns 的值设为  subgrid，grid-template-areas设为初始值
<grid-template-rows> / <grid-template-columns>：将 grid-template-columns 和  grid-template-rows 设置为相应地特定的值，并且设置grid-template-areas为none

```css
.container {
  grid-template: none | subgrid | <grid-template-rows> / <grid-template-columns>;
}
```
**示例：**
```css
.container {
  grid-template:
    [row1-start] "header header header" 25px [row1-end]
    [row2-start] "footer footer footer" 25px [row2-end]
    / auto 50px auto;
}
//等价于
.container {
  grid-template-rows: [row1-start] 25px [row1-end row2-start] 25px [row2-end];
  grid-template-columns: auto 50px auto;
  grid-template-areas: 
    "header header header" 
    "footer footer footer";
}
```


由于 <font color= "blue">grid-template </font>不会重置 隐式 网格属性（grid-auto-columns， grid-auto-rows， 和  grid-auto-flow），
这可能是你想在大多数情况下做的，建议使用*grid 属性* 而不是*grid-template*。

#### grid-column-gap / grid-row-gap
指定网格线(grid lines)的大小。你可以把它想象为设置列/行之间间距的宽度。

><line-size> ：长度值

```css
.container {
  grid-column-gap: <line-size>;
  grid-row-gap: <line-size>;
}
```

**示例：**
```css
.container {
  grid-template-columns: 100px 50px 100px;
  grid-template-rows: 80px auto 80px; 
  grid-column-gap: 10px;
  grid-row-gap: 15px;
}
```
![grid5](http://newimg88.b0.upaiyun.com/newimg88/2017/12/grid-column-row-gap.png)
只能在 列/行 之间创建间距，网格外部边缘不会有这个间距。

#### grid-gap

<font color= "blue">grid-column-gap</font> 和 <font color= "blue">grid-row-gap</font>的缩写语法

><grid-row-gap> <grid-column-gap>：长度值

```css
.container {
  grid-gap: <grid-row-gap> <grid-column-gap>;
}
```

**示例:**
如果grid-row-gap没有定义，那么就会被设置为等同于 grid-column-gap 的值。例如下面的代码是等价的：
```css
.container{
  grid-template-columns: 100px 50px 100px;
  grid-template-rows: 80px auto 80px; 
  grid-gap: 10px 15px;
}
//等价于
.container{
  /* 设置 [`grid-column-gap`](http://www.css88.com/archives/8510#prop-grid-column-row-gap) 和 [`grid-row-gap`](http://www.css88.com/archives/8510#prop-grid-column-row-gap) */  
  grid-column-gap: 10px;
  grid-column-gap: 10px; 
 
  /* 等价于 */  
  grid-gap: 10px 10px;
 
  /* 等价于 */  
  grid-gap: 10px;
}
```

#### justify-items

沿着 行轴线(row axis) 对齐 网格项(grid items) 内的内容（相反的属性是 align-items 沿着列轴线对齐）。该值适用于容器内的所有网格项。

>start：将内容对齐到网格区域(grid area)的左侧
end：将内容对齐到网格区域的右侧
center：将内容对齐到网格区域的中间（水平居中）
stretch：填满网格区域宽度（默认值）

```css
.container {
    justify-items: start | end | center | stretch;
}
```
#### align-items
沿着 列轴线(column axis) 对齐 网格项(grid items) 内的内容（相反的属性是 justify-items 沿着行轴线对齐）。该值适用于容器内的所有网格项。

>start：将内容对齐到网格区域(grid area)的顶部
end：将内容对齐到网格区域的底部
center：将内容对齐到网格区域的中间（垂直居中）
stretch：填满网格区域高度（默认值）

```css
.container {
  align-items: start | end | center | stretch;
}
```

#### justify-content
有时，你的网格合计大小可能小于其 网格容器(grid container) 大小。 如果你的所有 网格项(grid items) 都使用像 px 这样的非灵活单位设置大小，在这种情况下，您可以设置网格容器内的网格的对齐方式。 此属性沿着 行轴线(row axis) 对齐网格（相反的属性是 align-content ，沿着列轴线对齐网格）。

>start：将网格对齐到 网格容器(grid container) 的左边
end：将网格对齐到 网格容器 的右边
center：将网格对齐到 网格容器 的中间（水平居中）
stretch：调整 网格项(grid items) 的宽度，允许该网格填充满整个 网格容器 的宽度
space-around：在每个网格项之间放置一个均匀的空间，左右两端放置一半的空间
space-between：在每个网格项之间放置一个均匀的空间，左右两端没有空间
space-evenly：在每个栅格项目之间放置一个均匀的空间，左右两端放置一个均匀的空间

```css
.container {
  justify-content: start | end | center | stretch | space-around | space-between | space-evenly;    
}
```

![justify-content: start](http://newimg88.b0.upaiyun.com/newimg88/2017/12/grid-justify-content-start.png)

#### align-content

有时，你的网格合计大小可能小于其 网格容器(grid container) 大小。 如果你的所有 网格项(grid items) 都使用像 px 这样的非灵活单位设置大小，在这种情况下，您可以设置网格容器内的网格的对齐方式。 此属性沿着 列轴线(column axis) 对齐网格（相反的属性是 justify-content ，沿着行轴线对齐网格）。

>start：将网格对齐到 网格容器(grid container) 的顶部
end：将网格对齐到 网格容器 的底部
center：将网格对齐到 网格容器 的中间（垂直居中）
stretch：调整 网格项(grid items) 的高度，允许该网格填充满整个 网格容器 的高度
space-around：在每个网格项之间放置一个均匀的空间，上下两端放置一半的空间
space-between：在每个网格项之间放置一个均匀的空间，上下两端没有空间
space-evenly：在每个栅格项目之间放置一个均匀的空间，上下两端放置一个均匀的空间

```css
.container {
  align-content: start | end | center | stretch | space-around | space-between | space-evenly;  
}
```
![align-content: start](http://newimg88.b0.upaiyun.com/newimg88/2017/12/grid-align-content-start.png)

#### grid-auto-columns / grid-auto-rows

指定任何自动生成的网格轨道(grid tracks)（又名隐式网格轨道）的大小。在你明确定位的行或列（通过  grid-template-rows / grid-template-columns），超出定义的网格范围时，隐式网格轨道被创建了。

><track-size>：可以是长度值，百分比，或者等份网格容器中可用空间（使用 fr 单位）

```css
.container {
  grid-auto-columns: <track-size> ...;
  grid-auto-rows: <track-size> ...;
}
```
为了说明如何创建隐式网格轨道，请考虑一下以下的代码：

```css
.container {
  grid-template-columns: 60px 60px;
  grid-template-rows: 90px 90px
}
```
![grid7](http://newimg88.b0.upaiyun.com/newimg88/2017/12/grid-auto.png)

但现在想象一下，你使用 grid-column 和 grid-row 来定位你的网格项（grid items），像这样：
```css
.item-a {
  grid-column: 1 / 2;
  grid-row: 2 / 3;
}
.item-b {
  grid-column: 5 / 6;
  grid-row: 2 / 3;
}
```
![grid8](http://newimg88.b0.upaiyun.com/newimg88/2017/12/implicit-tracks.png)

因为我们引用的网格线不存在，所以创建宽度为 0 的隐式网格轨道以填补空缺。我们可以使用  grid-auto-columns 和 grid-auto-rows 来指定这些隐式轨道的大小：

```css
.container {
  grid-auto-columns: 60px;
}
```
![grid9](http://newimg88.b0.upaiyun.com/newimg88/2017/12/implicit-tracks-with-widths.png)

#### grid-auto-flow
如果你有一些没有明确放置在网格上的网格项(grid items)，自动放置算法 会自动放置这些网格项。该属性控制自动布局算法如何工作。
>row：告诉自动布局算法依次填充每行，根据需要添加新行
column：告诉自动布局算法依次填入每列，根据需要添加新列
dense：告诉自动布局算法在稍后出现较小的网格项时，尝试填充网格中较早的空缺

```css
.container {
  grid-auto-flow: row | column | row dense | column dense
}
```

#### grid
在一个声明中设置所有以下属性的简写： grid-template-rows, grid-template-columns,  grid-template-areas, grid-auto-rows, grid-auto-columns, 和 grid-auto-flow 。它还将grid-column-gap 和 grid-column-gap设置为初始值，即使它们不可以通过grid属性显式地设置。

>none：将所有子属性设置为其初始值
<grid-template-rows> / <grid-template-columns>：将 grid-template-rows 和  grid-template-columns 分别设置为指定值，将所有其他子属性设置为其初始值
<grid-auto-flow> [<grid-auto-rows> [ / <grid-auto-columns>] ] ：分别接受所有与  grid-auto-flow ，grid-auto-rows 和 grid-auto-columns 相同的值。如果省略了  grid-auto-columns ，它被设置为由 grid-auto-rows 指定的值。如果两者都被省略，他们就会被设置为初始值

```css
.container {
    grid: none | <grid-template-rows> / <grid-template-columns> | <grid-auto-flow> [<grid-auto-rows> [/ <grid-auto-columns>]];
}
```
**示例：**
```css
.container {
  grid: 200px auto / 1fr auto 1fr;
}
//等价于
.container {
  grid-template-rows: 200px auto;
  grid-template-columns: 1fr auto 1fr;
  grid-template-areas: none;
}

.container {
  grid: column 1fr / auto;
}
//等价于
.container {
  grid-auto-flow: column;
  grid-auto-rows: 1fr;
  grid-auto-columns: auto;
}

```

它也接受一个更复杂但相当方便的语法来一次设置所有内容。您可以  grid-template-areas，grid-template-rows和grid-template-columns，并所有其他的子属性都被设置为它们的初始值。这么做可以在它们网格区域内相应地指定网格线名字和网格轨道的大小。用最简单的例子来描述：

```css
.container {
  grid: [row1-start] "header header header" 1fr [row1-end]
        [row2-start] "footer footer footer" 25px [row2-end]
        / auto 50px auto;
}
//等价于

.container {
  grid-template-areas: 
    "header header header"
    "footer footer footer";
  grid-template-rows: [row1-start] 1fr [row1-end row2-start] 25px [row2-end];
  grid-template-columns: auto 50px auto;    
}
```

### 网格项(Grid Item)
网格容器（Grid Container）的子元素（例如直接子元素）。这里 item 元素就是网格项(Grid Item)，但是 sub-item 不是。
```html
<div class="container">
  <div class="item"></div> 
  <div class="item">
    <p class="sub-item"></p>
  </div>
  <div class="item"></div>
</div>
```

#### grid-column-start / grid-column-end / grid-row-start / grid-row-end
通过指定 网格线(grid lines) 来确定网格内 网格项(grid item) 的位置。 grid-column-start /  grid-row-start 是网格项目开始的网格线，grid-column-end / grid-row-end 是网格项结束的网格线。

><line> ：可以是一个数字引用一个编号的网格线，或者一个名字来引用一个命名的网格线
span <number> ：该网格项将跨越所提供的网格轨道数量
span <name> ：该网格项将跨越到它与提供的名称位置
auto ：表示自动放置，自动跨度，默认会扩展一个网格轨道的宽度或者高度

```css
.item {
    grid-column-start: <number> | <name> | span <number> | span <name> | auto
    grid-column-end: <number> | <name> | span <number> | span <name> | auto
    grid-row-start: <number> | <name> | span <number> | span <name> | auto
    grid-row-end: <number> | <name> | span <number> | span <name> | auto
}
```
**示例：**
```css
.item-a {
    grid-column-start: 2;
    grid-column-end: five;
    grid-row-start: row1-start
    grid-row-end: 3
}
```
![grid10](http://newimg88.b0.upaiyun.com/newimg88/2017/12/grid-start-end-a.png)

```css
.item-b {
    grid-column-start: 1;
    grid-column-end: span col4-start;
    grid-row-start: 2
    grid-row-end: span 2
}
```
![grid11](https://cdn.css-tricks.com/wp-content/uploads/2016/11/grid-start-end-b.png)
如果没有声明指定 grid-column-end / grid-row-end，默认情况下，该网格项将占据1个轨道。

项目可以相互重叠。您可以使用 z-index 来控制它们的重叠顺序。

#### grid-column / grid-row
分别为 grid-column-start + grid-column-end 和 grid-row-start + grid-row-end 的缩写形式。

><start-line> / <end-line>：每个网格项都接受所有相同的值，作为普通书写的版本，包括跨度

```css
.item {
    grid-column: <start-line> / <end-line> | <start-line> / span <value>;
    grid-row: <start-line> / <end-line> | <start-line> / span <value>;
}
```

**示例：**
```css
.item-c {
    grid-column: 3 / span 2;
    grid-row: third-line / 4;
}
```
![grid12](http://newimg88.b0.upaiyun.com/newimg88/2017/12/grid-start-end-c.png)
如果没有声明分隔线结束位置，则该网格项默认占据 1 个网格轨道。

#### grid-area
为网格项提供一个名称，以便可以 被使用网格容器 grid-template-areas 属性创建的模板进行引用。 另外，这个属性可以用作grid-row-start + grid-column-start + grid-row-end +  grid-column-end 的缩写。

> <name>：你所选的名称
<row-start> / <column-start> / <row-end> / <column-end>：数字或分隔线名称

```css
.item {
    grid-area: <name> | <row-start> / <column-start> / <row-end> / <column-end>;
}
```

#### justify-self
沿着 行轴线(row axis) 对齐 网格项 内的内容（ 相反的属性是 align-self ，沿着列轴线(column axis)对齐）。此值适用于单个网格项内的内容。

>start：将内容对齐到网格区域的左侧
end：将内容对齐到网格区域的右侧
center：将内容对齐到网格区域的中间（水平居中）
stretch：填充整个网格区域的宽度（这是默认值）

**示例：**
```css
.item-a {
    justify-self: start;
}
```
![justify-self: start](http://newimg88.b0.upaiyun.com/newimg88/2017/12/grid-justify-self-start.png)

#### align-self
沿着 列轴线(column axis) 对齐 网格项 内的内容（ 相反的属性是 justify-self ，沿着 行轴线(row axis) 对齐）。此值适用于单个网格项内的内容。
>start：将内容对齐到网格区域的顶部
end：将内容对齐到网格区域的底部
center：将内容对齐到网格区域的中间（垂直居中）
stretch：填充整个网格区域的高度（这是默认值）
```css
.item{
  align-self: start | end | center | stretch;
}
```

要为网格中的所有网格项设置 列轴线(column axis) 上的对齐方式，也可以在 网格容器 上设置  align-items 属性。

### 网格线(Grid Line)
构成网格结构的分界线。它们既可以是垂直的（“列网格线(column grid lines)”），也可以是水平的（“行网格线(row grid lines)”），并位于行或列的任一侧。例如，这里的黄线就是一条列网格线。
![grid2](http://newimg88.b0.upaiyun.com/newimg88/2017/12/grid-line.png)


### 网格轨道(Grid Track)
两条相邻网格线之间的空间。你可以把它们想象成网格的列或行。下图是第二条和第三条 行网格线 之间的 网格轨道(Grid Track)。
![grid3](http://newimg88.b0.upaiyun.com/newimg88/2017/12/grid-track.png)

### 网格单元格(Grid Cell)
两个相邻的行和两个相邻的列网格线之间的空间。这是 Grid(网格) 系统的一个“单元”。下图是第1至第2条 行网格线 和第2至第3条 列网格线 交汇构成的 网格单元格(Grid Cell)。
![grid4](http://newimg88.b0.upaiyun.com/newimg88/2017/12/grid-cell.png)

### 网格区域(Grid Area)
4条网格线包围的总空间。一个 网格区域(Grid Area) 可以由任意数量的 网格单元格(Grid Cell) 组成。下图是 行网格线1和3，以及列网格线1和3 之间的网格区域。
![grid5](http://newimg88.b0.upaiyun.com/newimg88/2017/12/grid-area.png)

更多关于 CSS Grid 布局的优秀文章
[5分钟学会 CSS Grid 布局](http://www.css88.com/archives/8506)
[CSS Grid 布局完全指南(图解 Grid 详细教程)](http://www.css88.com/archives/8510)
[如何使用 CSS Grid 快速而又灵活的布局](http://www.css88.com/archives/8512)