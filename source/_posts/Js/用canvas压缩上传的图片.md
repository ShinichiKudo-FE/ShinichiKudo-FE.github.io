---
title: 用canvas压缩上传的图片
date: 2018-08-07 15:38:38
tags: "Js"
categories: "Js"
---
# 前言

现在摄像头已经是手机的标配了，移动网站也做得越来越像APP。然而拍照上传这件事情的体验似乎仍然不如APP，主要原因是现在手机拍摄的照片太大，上传非常消耗流量也非常耗时。APP都会在上传前缩小要上传的照片尺寸，以期更节省流量和时间。在HTML5时代，利用文件API和Canvas技术，Web上也可以做到图片压缩上传。

## 过滤文件类型

首先我们希望用户能直接选择手机照片，而不是在各种类型的文件中选择。只需要在`input`标签中加入`accept`属性就可以实现这一点：

```html
<div id="preview"></div>
<form>
    <input type="file" accept="image/*">
    <input type="submit">
<form>
```

在`Android4以上，iOS7以上`设备实测，当用户点击这个文件选择器的时候，手机会自动调出图片库，并带有拍照选择。

## 读文件生成预览

用户选择了图片之后，需要读取文件内容，读出的内容可供生成预览图，也可以供后面压缩使用。使用HTML5的[FileReader API](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader)可以达成这一目的。

```js
var file = document.querySeletor("[type=file]");
file.addEventListener('change',function(e){
    for(var i = 0,f; f < e.target.file[i]; i++){
        if(f.type.indexOf('image') != 0) continue;
        var render = new FileReader();
        render.onload = function(e){
            var img = document.createElement('img');
            img.src = e.targte.result;
            document.getElementById('preview').appendChild(img);
        }
        render.readAsDataURL(f)
    }
},false)
```

<!-- more -->
如果不需要预览图，可以不把img对象添加到DOM上。

## 压缩

利用Canvas渲染上下文的[drawImage](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/drawImage)接口，可以把一张图片绘制到Canvas上，在这个过程中可以重新定义图片尺寸，然后再用Canvas的[toDataURL](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement/toDataURL)接口可以生成出压缩后的图片。

```js
var images = document.querySeletorAll('#preview img');
var iWidth = 400,iHeight = 300;
var compressedImages = [];
[].forEach.call(images,function(image){
    var canvas = document.createElement('canvas');
    canvas.width = iWidth;
    canvas.height = iHeight;
    canvas.getContent('2d').drawImage(image);
    var compressed = canvas.toDataURL("imgae/jepg",0.7);
    compressedImages.push(compressed);
})
```

## 上传

上面一步骤生成的压缩后图片是Data URL形式的，上传前需要把开头部分的`data:image/jpeg;base64`,截掉才是图片的`Base64`编码形式。

可以直接把`Base64`的字符串上传到服务器，然后由服务端解码为JPG图片，也可以在前端解码上传。如果要在前端解码并以文件方式上传，先要用[atob](https://developer.mozilla.org/zh-CN/docs/Web/API/WindowBase64/atob)函数把`Base64`解开，然后转换为[ArrayBuffer](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)，再用它创建一个[Blob](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)对象。文件方式上传需要用`multipart/form-data`格式，可以利用[FormData](https://developer.mozilla.org/zh-CN/docs/Web/Guide/Using_FormData_Objects)对象组装生成好的Blob对象来实现。

```js
function b64toBlob(b64Data,contentType,sliceSize){
    contentType = contentType || '';
    sliceSize = sliceSize || 512;

    var byteCharacters = atob(b64Data);
    var byteData = [];
    for(var offset = 0; offset < byteCharacters.length; offset += sliceSize){
        var slice = byteCharacters.slice(offset, offset + sliceSize);

        var byteNumbers = new Array(slice.length);
        for (var i = 0; i < slice.length; i++) {
            byteNumbers[i] = slice.charCodeAt(i);
        }

        var byteArray = new Uint8Array(byteNumbers);

        byteArrays.push(byteArray);
    }
    var blob = new Blob(byteArray,{type:content})
    return blob;
}

var fileBlob = b64toBlob(compressed.substr(23),'images/jepg');
var formData = new FormData();
formData.append('file',fileBlob);
var xhr = new XMLHttpRequest();
xhr.open('POST','upload.php');
xhr.send(formData);

```

## 安全

最后，请不要忘记在服务端对用户上传的文件数据进行合法性检查，毕竟黑客也可能用非浏览器上传垃圾文件或者恶意脚本。

## Demo

![扫描查看](https://lib.xts.so/xts-img/x/2017/02/3037522307.png)