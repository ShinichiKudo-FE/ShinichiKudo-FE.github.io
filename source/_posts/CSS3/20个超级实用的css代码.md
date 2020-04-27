---
title: 20个超级实用的css代码
date: 2018-03-18 22:46:04
tags: "Css3"
categories: "Css3"
---
[原文地址](https://www.w3cplus.com/css/20-incredibly-useful-CSS-snippets-for-developers)

大家非常的清楚CSS是我们Web制作中不可或缺的一部分。HTML提供了Web制作的结构，但他不能让我们实现美丽的页面制作，此时我们需要CSS的帮助。CSS虽然能帮我们完善Web制作的效果，但其在不同的浏览器下是有不可预知的效果，为了让你的CSS能解决这些不一致下，今天给大家介绍25个CSS技巧代码，我相信这些对你肯家有很大的作用。

## 使用text-indent来隐藏文本
这个常用在图片替换文本中，最常见的就是使用使用图片来替换Logo，这个是非常有用的，使用“text-indent”我们可以达到图片替换文本的效果，而且方便搜索引擎的优化，还能支持阅读器阅读网页内容：
```css
			h1 {
				text-indent:-9999px;
				margin:0 auto;
				width:400px;
				height:100px;
				background:transparent url("images/logo.jpg") no-repeat scroll;
			}
```	
## 根据文件格式设置链接图标
这个技巧主要是针对用户体验，让用户能很清楚点击的链接是有关于什么方面的内容，比如说，点击某个链接会到跳到站外。换句话说使用属性选择器器，给不同的链接设置不同的图标，让用户很轻松的明白相应的链接是有关于什么方面的内容：
```css
			a[href^="http:"] {
					display:inline-block;
					padding-right:14px;
					background:transparent url(/Images/ExternalLink.gif) center right no-repeat;
			}
			a[href^="mailto:"] {
					display:inline-block;
					padding-left:20px;
					line-height:18px;
					background:transparent url(/Images/MailTo.gif) center left no-repeat;
			}
			a[href$='.pdf'] {
					display:inline-block;
					padding-left:20px;
					line-height:18px;
					background:transparent url(/Images/PDFIcon.gif) center left no-repeat;
			}
			a[href$='.swf'], a[href$='.fla'], a[href$='.swd'] {
					display:inline-block;
					padding-left:20px;
					line-height:18px;
					background:transparent url(/Images/FlashIcon.gif) center left no-repeat;
			}
			a[href$='.xls'], a[href$='.csv'], a[href$='.xlt'], a[href$='.xlw'] {
					display:inline-block;
					padding-left:20px;
					line-height:18px;
					background:transparent url(/Images/ExcelIcon.gif) center left no-repeat;
			}
			a[href$='.ppt'], a[href$='.pps'] {
					display:inline-block;
					padding-left:20px;
					line-height:18px;
					background:transparent url(/Images/PowerPointIcon.gif) center left no-repeat;
			}
			a[href$='.doc'], a[href$='.rtf'], a[href$='.txt'], a[href$='.wps'] {
					display:inline-block;
					padding-left:20px;
					line-height:18px;
					background:transparent url(/Images/WordDocIcon.gif) center left no-repeat;
			}
			a[href$='.zip'], a[href$='.gzip'], a[href$='.rar'] {
					display:inline-block;
					padding-left:20px;
					line-height:18px;
					background:transparent url(/Images/ZIPIcon.gif) center left no-repeat;
			}
		
```

## 在IE浏览器中删除textarea的滚动条
IE浏览器中textarea默认就有滚动条出现，为了达到所有浏览器默认下一致的效果，其实我们可以使用代码让他达到一致的效果：

```css
textarea{
    overflow:auto;
}
```

## 段落首字下沉
有杂志排版中我们常看到第一个段落的首字下沉的效果，其实这种效果实现是相当的容易：
```css
    p:first-letter{
        display:block;
        margin:5px 0 0 5px;
        float:left;
        color:#FF3366;
        font-size:60px;
        font-family:Georgia;
    }
```

## 所有浏览器下的CSS透明度
```css
    .transparent {
        zoom: 1;
        filter: alpha(opacity=50);
        opacity: 0.5;
    }
```

但是使用opacity会影响其后代元素的透明度，我们可以考虑使用：
```css
		.transparent {
			/* Fallback for web browsers that doesn't support RGBa */
			background: rgb(0, 0, 0);
			/* RGBa with 0.6 opacity */
			background: rgba(0, 0, 0, 0.6);
			/* For IE 5.5 - 7*/
			filter:progid:DXImageTransform.Microsoft.gradient(startColorstr=#99000000, endColorstr=#99000000);
			/* For IE 8*/
			-ms-filter: "progid:DXImageTransform.Microsoft.gradient(startColorstr=#99000000, endColorstr=#99000000)";
		}
```	

## Reset Css
```css
body,ul,ol,dl,dd,dir,h1,h2,h3,h4,h5,h6,p,pre,blockquote,hr,figure{
      margin:0;
      padding:0;
      font-size: 100%;
      vertical-align:baseline;
		}

		table {
			border-collapse:collapse;
			border-spacing:0;
		}

		article, aside, dialog, figure, footer, header, hgroup, nav, section { 
			display:block;
		}

		/*
		 * Corrects inline-block display not defined in IE6/7/8/9 & FF3
		 */

		audio,
		canvas,
		video {
			display: inline-block;
			*display: inline;
			*zoom: 1;
		}

		/*
		 * Prevents modern browsers from displaying 'audio' without controls
		 */

		audio:not([controls]) {
			display: none;
		}

		/*
		 * Addresses styling for 'hidden' attribute not present in IE7/8/9, FF3, S4
		 * Known issue: no IE6 support
		 */

		[hidden] {
			display: none;
		}
```

## 图片预加载
```css
#preloader {
			/* Images you want to preload*/
			background-image: url(image1.jpg);
			background-image: url(image2.jpg);
			background-image: url(image3.jpg);
			width: 0px;
			height: 0px;
			display: inline;
		}
```

## 基本的CSS Sprite按钮

```css
a {
    display: block;
    background: url(sprite.png) no-repeat;
    height: 30px;
    width: 250px;
    }

a:hover {
    background-position: 0 -30px;
}
```

## Google Font API
```css
<head>
    Inconsolata:italic|Droid+Sans"
</head>

body {
    font-family: 'Tangerine', 'Inconsolata', 'Droid Sans', serif; font-size: 48px;
}
```

## 浏览器的专用hack
浏览器的兼容问题向来都是很烦的事情，特别是在IE下的兼容问题。但有时我们为了达到一致的效果，不得不使用浏览器的兼容：
```css
		/* IE 6 */
		* html .yourclass { }

		/* IE 7 */
		*+html .yourclass{ }

		/* IE 7 and modern browsers */
		html>body .yourclass { }

		/* Modern browsers (not IE 7) */
		html>/**/body .yourclass { }

		/* Opera 9.27 and below */
		html:first-child .yourclass { }

		/* Safari */
		html[xmlns*=""] body:last-child .yourclass { }

		/* Safari 3+, Chrome 1+, Opera 9+, Fx 3.5+ */
		body:nth-of-type(1) .yourclass { }

		/* Safari 3+, Chrome 1+, Opera 9+, Fx 3.5+ */
		body:first-of-type .yourclass {  }

		/* Safari 3+, Chrome 1+ */
		@media screen and (-webkit-min-device-pixel-ratio:0) {
		 .yourclass  {  }
		}
 ```       

## 固定页脚
固定页脚在屏幕的底部，在现代浏览器来说是一件非常容易的事情，但是在IE6下还是需要特殊的处理：
```css
		#footer {
			position:fixed;
			left:0px;
			bottom:0px;
			height:30px;
			width:100%;
			background:#999;
		}

		/* IE 6 */
		* html #footer {
			position:absolute;
			top:expression((0-(footer.offsetHeight)+(document.documentElement.clientHeight ? document.documentElement.clientHeight : document.body.clientHeight)+(ignoreMe = document.documentElement.scrollTop ? document.documentElement.scrollTop : document.body.scrollTop))+'px');
		}
```	

## 翻转图片
翻转图像随着CSS3的transform越来越实用，不需要重新加载图片，就可以实现一个图片的旋转。常见的是一个三角型效果，我们想让他在不同状态展示不同的风格：
```css
    img.flip {
        -moz-transform: scaleX(-1);
        -o-transform: scaleX(-1);
        -webkit-transform: scaleX(-1);
        transform: scaleX(-1);
        filter: FlipH;
        -ms-filter: "FlipH";
    }
```

## clearfix
clearfix主要是使用他来清除浮动，只需要添加这个类名无需加上任何HTML标记，就可以达到清除浮动的效果：
```css
		.clearfix:before,
		.clearfix:after {
			content: " ";
			display:table;
		}

		.clearfix:after {
			clear:both;
			overflow:hidden;
		}
		.clearfix { 
			zoom: 1;
		}
```

## 圆角
随着CSS3的属性的出现，我们制作圆角效果就不需要在像以前那样的辛苦了，可以使用CSS3的border-radius来实现，只是在IE-6-8下无法实现，我们来看现代浏览器下如何制作圆角：
```css
		.round{
			-moz-border-radius: 10px;
			-webkit-border-radius: 10px;
			-khtml-border-radius: 10px; /* for old Konqueror browsers */
			border-radius: 10px; /* future proofing */
		}
```

## !important
!important有时可以帮我们做很多事，他可以覆盖任何相同的样式，换句话说他可以改为样式的权重：
```css
		p{
			font-size:20px !important;
		}
```

## @font-face
@font-face也是CSS3的属性之一，他能在所有浏览器下运行。最大的作用就是让用户没有字体的浏览下也能支持网页字体，具体使用：
```css
		@font-face {
			font-family: 'Graublau Web';
			src: url('GraublauWeb.eot');
			src: local('☺'),
				url('GraublauWeb.woff') format('woff'), url('GraublauWeb.ttf') format('truetype');
		}
		h2 {
			font-family:'Graublau Web';
		}
```	

## 页面水平居中
如何使一个网站的页面水平居中显示，我想这个不用我说大家也知道，因为大家肯定使用过多次了。
```css
		.wrapper {
			width:960px;
			margin:0 auto;
		}
```

## 最小高度min-height
在IE6浏览器下是不支持最小高度这个属性的，为了解决这个问题，我们可以使用下面这样的代码来处理：
```css
		.box {
			min-height:500px;
			height:auto !important;
			height:500px;
		}
```	

## 垂直居中
水平居中处理起来相当的简单的，但是垂直居中处理起来还是相当的烦，特别是要兼容IE的浏览器情况下：
```css
		div {
			height: 100px;
			line-height: 100px;
			white-space: nowrap;
		}

		img { vertical-align: middle; }

		.for_ie6 { display: inline-block; }
		.for_ie6 { display: inline; }
```

## ::selection
有很多朋友肯定不知道这个属性的作用。它可以改变选择的文本的背景色和前景色，突出你的浏览器中的选择文本效果：
```css
		::selection {
			color: #000000;
			background-color: #FF0000;
		}
		::-moz-selection {
			color: #000000;
			background: #FF0000;
		}
```	
