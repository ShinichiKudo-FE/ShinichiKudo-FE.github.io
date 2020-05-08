---
title: websocket断开原因分析
tags: 'websocket'
categories: 'HTTP'
date: 2020-04-28 17:07:15
---
[原文地址](https://segmentfault.com/a/1190000014582485)
# 把错误打印出来

WebSocket断开的原因有很多，最好在WebSocket断开时，将错误打印出来。

![demo](https://segmentfault.com/img/remote/1460000014582490?w=812&h=408)

```js
ws.onclose = function (e) {
  console.log('websocket 断开: ' + e.code + ' ' + e.reason + ' ' + e.wasClean)
  console.log(e)
}
```

如果你想自己玩玩`WebSocket`, 但是你又不想自己部署一个`WebSocket`服务器，你可以使用`ws = new WebSocket('wss://echo.websocket.org/')`, 你向echo.websocket.org发送消息，它会回复你同样的消息。

# 重要信息错误状态码

`WebSocket`断开时，会触发`CloseEvent`, `CloseEvent`会在连接关闭时发送给使用 `WebSockets` 的客户端. 它在 `WebSocket` 对象的 onclose 事件监听器中使用。`CloseEvent`的code字段表示了`WebSocket`断开的原因。可以从该字段中分析断开的原因。

`CloseEvent`有三个字段需要注意, 通过分析这三个字段，一般就可以找到断开原因
> * ` CloseEvent.code`: code是错误码，是整数类型
> * `CloseEvent.reason`: reason是断开原因，是字符串
> * `CloseEvent.wasClean`: wasClean表示是否正常断开，是布尔值。一般异常断开时，该值为false

![closed-code](https://segmentfault.com/img/remote/1460000014074898?w=1768&h=732)

# 关闭状态码表

一般来说`1006`的错误码出现的情况比较常见，该错误码一般出现在断网时。

| 状态码        | 名称   | 描述  |
| --------   | -----:  | :----:  |
| 0–999 |  |   保留段, 未使用     |
| 1000 |  CLOSE_NORMAL   |  正常关闭; 无论为何目的而创建, 该链接都已成功完成任务.   |
| 1001 |  CLOSE_GOING_AWAY  |  终端离开, 可能因为服务端错误, 也可能因为浏览器正从打开连接的页面跳转离开.  |
| 1002 | CLOSE_PROTOCOL_ERROR | 由于协议错误而中断连接.|
| 1003 | CLOSE_UNSUPPORTED | 由于接收到不允许的数据类型而断开连接 (如仅接收文本数据的终端接收到了二进制数据).|
| 1004 | | 保留. 其意义可能会在未来定义. |
| 1005 | |CLOSE_NO_STATUS|
| 1006 | CLOSE_ABNORMAL | 保留. 用于期望收到状态码时连接非正常关闭 (也就是说, 没有发送关闭帧). |
|1007|	Unsupported Data|	由于收到了格式不符的数据而断开连接 (如文本消息中包含了非 UTF-8 数据).|
|1008|	Policy Violation|	由于收到不符合约定的数据而断开连接. 这是一个通用状态码, 用于不适合使用 1003 和 1009 状态码的场景.|
|1009|	CLOSE_TOO_LARGE|	由于收到过大的数据帧而断开连接.|
|1010|	Missing Extension|	客户端期望服务器商定一个或多个拓展, 但服务器没有处理, 因此客户端断开连接.|
|1011|	Internal Error|	客户端由于遇到没有预料的情况阻止其完成请求, 因此服务端断开连接.|
|1012|	Service Restart|	服务器由于重启而断开连接.|
|1013|	Try Again Later|	服务器由于临时原因断开连接, 如服务器过载因此断开一部分客户端连接.|
|1014|		|由 WebSocket标准保留以便未来使用.|
|1015|	|TLS Handshake	保留. 表示连接由于无法完成 TLS 握手而关闭 (例如无法验证服务器证书).|
|1016–1999|		|由 WebSocket标准保留以便未来使用.|
|2000–2999|		|由 WebSocket拓展保留使用.|
|3000–3999|		|可以由库或框架使用.? 不应由应用使用. 可以在 IANA 注册, 先到先得.|
|4000–4999|		|可以由应用使用.|

# 其他注意事项

如果你的服务所在的域是HTTPS的，那么使用的WebSocket协议也必须是wss, 而不能是ws

