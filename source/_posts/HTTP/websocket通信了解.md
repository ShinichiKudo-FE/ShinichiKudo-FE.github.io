---
title: websocket通信了解
date: 2018-11-04 23:15:50
tags: "websocket"
categories: "HTTP"
---
# websocket

websocket是HTML5出的协议，他是一个持久化的协议。

## websocket是什么样的协议，有什么与优点？

首先websocket是一个持久化的协议，在HTTP中一个request只能有一个response,而且这个response也是被动的，不能主动发起。

下面是用websocket请求的header头

```js
GET /chat HTTP/1.1
Host: server.example.com

//告知服务器是使用websocket的协议
Upgrade: websocket   
Connection: Upgrade

Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw== //是一个Base64 encode的值，这个是浏览器随机生成的，验证服务器
Sec-WebSocket-Protocol: chat, superchat  // 是一个用户定义的字符串，用来区分同URL下，不同的服务所需要的协议
Sec-WebSocket-Version: 13 //  是告诉服务器所使用的Websocket Draft（协议版本）
Origin: http://example.com
```

下面是服务器的返回头

```js
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat
```

下面是HTTP告诉客户端，已经切换协议了

```js
Upgrade: websocket
Connection: Upgrade
```
<!-- more -->
##　websocket的作用

ajax轮询的原理很简单，每隔几秒发送一次请求，询问服务器是否有信息。

http long poll的原理与ajax轮询差不多，不过  采用的是阻塞模型（一直打电话，没收到就不挂电话）

这两者共同的地方都是在不断地建立HTTP连接，等待服务端处理，上面这两种都是非常消耗资源的。可以体现出HTTP的另外一个特点：**被动性**

其实我们所用的程序是要经过两层代理的，即HTTP协议在Nginx等服务器的解析下，然后再传送给相应的Handler（PHP等）来处理。
简单地说，我们有一个非常快速的接线员（Nginx），他负责把问题转交给相应的客服（Handler）。

Websocket **只需要一次HTTP握手，所以说整个通讯过程是建立在一次连接/状态中，也就避免了HTTP的非状态性**，服务端会一直知道你的信息，直到你关闭请求，这样就解决了接线员要反复解析HTTP协议，还要查看identity info的信息。

