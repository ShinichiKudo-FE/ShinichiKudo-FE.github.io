---
title: 后HTML5时代
date: 2018-02-23 10:14:14
tags: "HTML5"
categories: "HTML5"
---
# HTML5 Device API
[原文地址](http://tgideas.qq.com/webplat/info/news_version3/804/7104/7106/m5723/201506/354489.shtml)
## 让音乐随心而动 - 音频处理 Web audio API

Audio对象提供的只是音频文件的播放，而Web Audio则是给了开发者对音频数据进行分析、处理的能力，比如混音、过滤。

> 系统要求 :
ios6+、android chrome、android firefox

**核心代码** :

``` javascript
var context = new webkitAudioContext();
var source = context.createBufferSource();   // 创建一个声音源
source.buffer = buffer;   // 告诉该源播放何物 
createBufferSourcesource.connect(context.destination);   // 将该源与硬件相连
source.start(0); //播放

```
<!-- more -->
技术分析 :

当我们加载完音频数据后，我们将创建一个全局的AudioContext对象来对音频进行处理，AudioContext可以创建各种不同功能类型的音频节点AudioNode

### 1.源节点（source node）
#### 1.audio标签
``` javascript
var sound, audio = new Audio();
audio.addEventListener('canplay', function() {
 sound = context.createMediaElementSource(audio);
 sound.connect(context.destination);
});
audio.src = '/audio.mp3';

```
#### 2.XMLHttpRequest 
```javascript
var sound, context = createAudioContext();
var audioURl = '/audio.mp3'; // 音频文件URL
var xhr = new XMLHttpRequest();
xhr.open('GET', audioURL, true);
xhr.responseType = 'arraybuffer'; 
xhr.onload = function() {
 context.decodeAudioData(request.response, function (buffer) {
 source = context.createBufferSource();
 source.buffer = buffer;
 source.connect(context.destination);
 }
}
xhr.send();
```
### 2.分析节点（analyser node）
```javascript
var audioCtx = new (window.AudioContext || window.webkitAudioContext)();
var analyser = audioCtx.createAnalyser();
analyser.fftSize = 2048;
var bufferLength = analyser.frequencyBinCount;
var dataArray = new Uint8Array(bufferLength);
analyser.getByteTimeDomainData(dataArray);

function draw() {
 drawVisual = requestAnimationFrame(draw);
 analyser.getByteTimeDomainData(dataArray);
 // 将dataArray数据以canvas方式渲染出来
};

draw();
```

### 3.处理节点（gain node、panner node、wave shaper node、delay node、convolver node等）
不同的处理节点有不同的作用，比如使用BiquadFilterNode调整音色（大量滤波器）、使用ChannelSplitterNode分割左右声道、使用GainNode调整增益值实现音乐淡入淡出等等。需要了解更多的音频节点可能参考：
[https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)

### 4.目的节点（destination node）
所有被渲染音频流到达的最终地点

## 捕捉用户摄像头 - 媒体流 Media Capture

通过getUserMedia捕捉用户摄像头获取视频流和通过麦克风获取用户声音。

> 系统要求 :
android chrome、android firefox

**核心代码** :
### 1、摄像头捕捉
```javascript
navigator.webkitGetUserMedia ({video: true}, function(stream) {
 video.src = window.URL.createObjectURL(stream);
 localMediaStream = stream;
}, function(e){

})
```

### 2、从视频流中拍照
```javascript
btnCapture.addEventListener('touchend', function(){
	if (localMediaStream) {
		canvas.setAttribute('width', video.videoWidth);
		canvas.setAttribute('height', video.videoHeight);
		ctx.drawImage(video, 0, 0);
	}
}, false);
```

### 3、用户声音录制
```javascript
navigator.getUserMedia({audio:true}, function(e) {
	context = new audioContext();
	audioInput = context.createMediaStreamSource(e);	
	volume = context.createGain();
	recorder = context.createScriptProcessor(2048, 2, 2);
	recorder.onaudioprocess = function(e){
		recordingLength += 2048;
		recorder.connect (context.destination); 
	}	
}, function(error){});
```

### 4、保存用户录制的声音
```javascript
var buffer = new ArrayBuffer(44 + interleaved.length * 2);
var view = new DataView(buffer);
fileReader.readAsDataURL(blob); // android chrome audio不支持blob
… audio.src = event.target.result;
```

## 你是逗逼？ - 语音识别 Web Speech API
将文本转换成语音；将语音识别为文本。

> 系统要求 :
ios7+，android chrome，android firefox

**核心代码** :

### 1、文本转换成语音，使用SpeechSynthesisUtterance对象.
```javascript
var msg = new SpeechSynthesisUtterance();
var voices = window.speechSynthesis.getVoices();
msg.volume = 1; // 0 to 1
msg.text = ‘识别的文本内容’;
msg.lang = 'en-US';
speechSynthesis.speak(msg);
```

### 2、语音转换为文本，使用SpeechRecognition对象.
```javascript

var newRecognition = new webkitSpeechRecognition();
newRecognition.onresult = function(event){
	var interim_transcript = ''; 
	for (var i = event.resultIndex; i < event.results.length; ++i) {
		final_transcript += event.results[i][0].transcript;
	}
};
```

测试结论：
Android支持不稳定；语音识别测试失败（暂且认为是某些内置接口被墙所致）。



## 让我尽情呵护你 - 设备电量 Battery API
查询用户设备电量及是否正在充电。

> 系统要求 :
android firefox

**核心代码** ：
```javascript
var battery = navigator.battery || navigator.webkitBattery || navigator.mozBattery || navigator.msBattery;
var str = '';
if (battery) {
 str += '<p>你的浏览器支持HTML5 Battery API</p>';
 if(battery.charging) {
 str += '<p>你的设备正在充电</p>';
} else {
 str += '<p>你的设备未处于充电状态</p>';
}
 str += '<p>你的设备剩余'+ parseInt(battery.level*100)+'%的电量</p>';
} else {
 str += '<p>你的浏览器不支持HTML5 Battery API</p>';
}
```

测试结论 ：
1、QQ浏览器与UC浏览器支持该接口，但未正确显示设备电池信息；
2、caniuse显示android chrome42支持该接口，实测不支持。

## 获取用户位置 - 地理位置 Geolocation API
Geolocation API用于将用户当前地理位置信息共享给信任的站点，目前主流移动设备都能够支持。

> 系统要求 :
ios6+、android2.3+

**核心代码**：

```javascript
var domInfo = $("#info");

// 获取位置坐标
if (navigator.geolocation) {
	navigator.geolocation.getCurrentPosition(showPosition,showError);
}
else{
	domInfo.innerHTML="抱歉，你的浏览器不支持地理定位！";
}

// 使用腾讯地图显示位置
function showPosition(position) {
	var lat=position.coords.latitude;
	var lon=position.coords.longitude;

	mapholder = $('#mapholder')
	mapholder.style.height='250px';
	mapholder.style.width = document.documentElement.clientWidth + 'px';

	var center = new soso.maps.LatLng(lat, lon);
	var map = new soso.maps.Map(mapholder,{
		center: center,
		zoomLevel: 13
	});

	var geolocation = new soso.maps.Geolocation();
	var marker = null;
	geolocation.position({}, function(results, status) {
		console.log(results);
		var city = $("#info");
		if (status == soso.maps.GeolocationStatus.OK) {
			map.setCenter(results.latLng);
			domInfo.innerHTML = '你当前所在城市: ' + results.name;
		if (marker != null) {
			marker.setMap(null);
		}
		// 设置标记
		marker = new soso.maps.Marker({
			map: map,
			position:results.latLng
		});
		} else {
			alert("检索没有结果，原因: " + status);
		}
	});
}
```

测试结论：

1、Geolocation API的位置信息来源包括GPS、IP地址、RFID、WIFI和蓝牙的MAC地址、以及GSM/CDMS的ID等等。规范中没有规定使用这些设备的先后顺序。
2、初测3g环境下比wifi环境理定位更准确；
3、测试三星 GT-S6358(android2.3) geolocation存在，但显示位置信息不可用POSITION_UNAVAILABLE。

## 把用户捧在手心 - 环境光 Ambient Light API
Ambient Light API定义了一些事件，这些时间可以提供源于周围光亮程度的信息，这通常是由设备的光感应器来测量的。设备的光感应器会提取出辉度信息。

> 系统要求 :
android firefox

**核心代码**：
这段代码实现感应用前当前环境光强度，调整网页背景和文字颜色。

```javascript
var domInfo = $('#info');
if (!('ondevicelight' in window)) {
	domInfo.innerHTML = '你的设备不支持环境光Ambient Light API';
} else {
	var lightValue = document.getElementById('dl-value');
	window.addEventListener('devicelight', function(event) {
		domInfo.innerHTML = '当前环境光线强度为：' + Math.round(event.value) + 'lux';
		var backgroundColor = 'rgba(0,0,0,'+(1-event.value/100) +')';
		document.body.style.backgroundColor = backgroundColor;
		if(event.value < 50) {
			document.body.style.color = '#fff'
		} else {
			document.body.style.color = '#000'
		}
	});
}
```
## 陀螺仪 Deviceorientation
装逼说法：设备移动传感器。

通俗说法：检测设备运动及运动状况，可以通过该接口获取到设备运动的方向和速度等相关信息

**核心代码** :
这货其实就是一个事件，简单地通过事件监听就可以获取到相关的设备运动信息
```javascript
window.addEventListener("devicemotion", function(event) {}, false);
```
最简单的设备运动，我们基本上可以认为是前后、左右、上下这么6个方向上的，装换成3D环境中来讲的话，就是在x、y和z轴上运动，这三个轴上的信息，都包含在了event对象中。我们可以通过这种方式获得

```javascript
window.addEventListener("devicemotion", function(event) {
    var eventacceleration = event.acceleration;
    document.querySelector('#devicemotion').innerHTML = "acceleration:<br>"+
    eventacceleration.x+"<br>"+
    eventacceleration.y+"<br>"+
    eventacceleration.z
}, false);
```
不要问我为什么手机躺在桌面上，上面的数字还在不停地抖动，咱们文化人应该都知道什么叫相对静止。上面的三个不断变化的数字，就是你设备的运动数据了。如果你在惊讶为什么我都已经移动过手机，为什么上面的数字貌似也没什么大的变化~那么我会告诉你说：有本事你的眼睛跟着你的手机一样快速移动啊，运动过程中你就能发现这个值的变化了。没有这个本事吧！~自己尝试修改下那个demo。把运动中值的变化都记录下来看看。。。（PS：反正我不弄。）

还有一种比较理想的设备运动方式就是不产生位移，只是快速的旋转手机。这个旋转的信息同样也被反馈在了event对象中

```javascript
window.addEventListener("devicemotion", function(event) {
    eventrotationRate = event.rotationRate;
    document.querySelector('#devicemotion').innerHTML = 'rotationRate:<br>'+
    eventrotationRate.alpha+'<br>'+
    eventrotationRate.beta+"<br>"+
    eventrotationRate.gamma
}, false);
```
可能有的童鞋会觉得~有了这个旋转应该就能实现一些手机左右旋转产生视差的效果了，其实是不行的~因为这货和上面的一样，设备停止后，我们可以认为他的值归0~。真正想要实现手机旋转视差，我们需要用到的就是另外一个event的属性accelerationIncludingGravity。从字面上理解这个属性就是重力加速度。
```javascript
window.addEventListener("devicemotion", function(event) {
    eventaccelerationIncludingGravity = event.accelerationIncludingGravity;
    document.querySelector('#devicemotion').innerHTML = "accelerationIncludingGravity:<br>"+
    eventaccelerationIncludingGravity.x+"<br>"+
    eventaccelerationIncludingGravity.y+"<br>"+
    eventaccelerationIncludingGravity.z
}, false);
```
其实，devicemotion还有一个好基友，这里也推荐给大家看一眼吧：

**deviceorientation**
这个事件和devicemotion的使用方法基本一致
```javascript
window.addEventListener("deviceorientation", function(event) {
    document.querySelector('#deviceorientation').innerHTML = 
    event.alpha+'<br>'+
    event.beta+"<br>"+
    event.gamma+'<br>';
}, false);
```

有前面devicemotion好基友的帮忙，似乎能做的东西又可以更多一点了。咱们前面貌似提及过&ldquo;喝前摇一摇&rdquo;。摇完之后，自然是要打开瓶盖痛饮一番了，结合旋转加速度，我们是不是可以尝试些一个拧瓶盖比赛的游戏了？怕长胖？要不，拧个螺丝钉也行。
## Websocket
浏览器与服务器全双工通信(full-duplex)。

**核心代码** ：

```javascript
var ws = new WebSocket('ws://127.0.0.1:8808/');//建立服务器连接ws.onopen = function(){
	systemInfo.innerHTML = '<p>和websocket服务器连接成功</p>';
}//接收到服务器返回的数据ws.onmessage = function(e){
	systemInfo.innerHTML += '<p>'+e.data+'</p>';
}//断开服务器连接ws.onclose = function(){
	systemInfo.innerHTML += '<p>WebSocket服务器连接关闭</p>';
}//ws发生错误ws.onerror = function(e){
	console.log(e);
	systemInfo.innerHTML += '<p>WebSocket发生错误</p>';
}
```
 我们可以通过事件的监听，去实时获取到网络状态的变化
 ```javascript
window.addEventListener('offline', function(e) {alert('offline');})
window.addEventListener('online', function(e) {alert('online');})
 ```
## NFC
近场通信/近距离无线通信技术。

其实说实在的，这个功能我并没有在我的页面里面调试出来。主要的一个原因是~这货目前只是在firefox os系统里面的firefox浏览器里面实现了，手头上没这设备。不过从他们官网的例子中，我大概地可以感受得到这货的好用，有兴趣的同学自行前往学习：[https://developer.mozilla.org/en-US/docs/Web/API/NFC_API/Using_the_NFC_API]https://developer.mozilla.org/en-US/docs/Web/API/NFC_API/Using_the_NFC_API.
## 震动 - Vibration API
设备震动

**核心代码** :
```javascript
navigator.vibrate = navigator.vibrate ||  navigator.webkitVibrate || 
                                navigator.mozVibrate || navigator.msVibrate;
navigator.vibrate(value);
navigator.vibrate(0);
```
vibrate的参数为震动的时间，

如果值为0，说明停止震动

值为一个数组的话，奇数项表示的是震动时间，偶数项表示的为间隔时间。

比如：vibrate(1000,1000,2000)  就是表示震动1秒之后暂停1秒，再震动2秒
## 网络环境 Connection API