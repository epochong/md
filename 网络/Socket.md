---
wetitle: Socket
date: 2019-09-07 11:55:25
tags: JavaEE
---

> 本文主要介绍WebSocket通信原理，已经WebSocket编程用到的响应技术，进来看看吧

<!--more-->

# Socket编程

## Socket 基本概念

Socket 是对 TCP/IP 协议族的一种封装，Socket 其实并不是一个标准的协议，而是应用层与 TCP/IP 协议族通信的中间软件抽象层，它是一组接口，工作位置基本在 OSI 模型会话层（第5层），是为了方便大家直接使用更底层协议（一般是 TCP 或 UDP ）而存在的一个抽象层。从设计模式的角度看来，Socket其实就是一个门面模式（facade），它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

Socket 还可以认为是一种网络间不同计算机上的进程通信的一种方法，利用三元组（ip地址，协议，端口）就可以唯一标识网络中的进程，网络中的进程通信可以利用这个标志与其它进程进行交互。

Socket 起源于 Unix ，Unix/Linux 基本哲学之一就是“一切皆文件”，都可以用“打开(open) –> 读写(write/read) –> 关闭(close)”模式来进行操作。因此 Socket 也被处理为一种特殊的文件。

![image-20200806205212579](/Users/wangchong/Library/Application Support/typora-user-images/image-20200806205212579.png)

最早的一套 Socket API 是采用 C 语言实现的，也就成为了 Socket 的事实标准，常见的 Socket API 实现：

![image-20200806205244922](/Users/wangchong/Library/Application Support/typora-user-images/image-20200806205244922.png)

# WebSocket

### 一. 介绍

#### 1. WebSocket简介

- WebSocket 是一种标准协议，用于在客户端和服务端之间进行双向数据传输。它是基于 TCP 的一种独立实现，支持长连接，**WebSocket**是应用**层**协议。
- 以前客户端想知道服务端的处理进度，要不停地使用 Ajax 进行轮询（即采用http协议不断发送请求），让浏览器隔个几秒就向服务器发一次请求，浪费流量（http请求头比较大）、浪费资源（没有更新也要请求）、消耗服务器CPU占用（没有信息也要接收请求），这对服务器压力较大。另外一种轮询就是采用 long poll 的方式，这就跟打电话差不多，没收到消息就一直不挂电话，也就是说，客户端发起连接后，如果没消息，就一直不返回 Response 给客户端，连接阶段一直是阻塞的。
- 而 WebSocket 解决了 HTTP 的这几个难题。当服务器完成协议升级后（ HTTP -> WebSocket ），服务端可以主动推送信息给客户端，解决了轮询造成的同步延迟问题。由于 WebSocket 只需要一次 HTTP 握手，服务端就能一直与客户端保持通信，直到关闭连接，这样就解决了服务器需要反复解析 HTTP 协议，减少了资源的开销，并且服务端和客户端之间的标头信息很小，可以降低服务端的资源浪费。
- HTML5 给 Web 浏览器带来了全双工 TCP 连接 websocket 标准服务器的能力。
- 换句话说，浏览器能够与服务器建立连接，通过已建立的通信信道来发送和接收数据而不需要由 HTTP 协议引入额外其他的开销来实现。

#### 2. WebSocket握手

客户端和服务器端 TCP 连接建立在 HTTP 协议握手发生之后。通过 HTTP 流量调试，很容易观察到握手。客户端一创建一个 WebSocket。

##### **客户端请求报文：**

> GET / HTTP/1.1
> Upgrade: websocket
> Connection: Upgrade
> Host: example.com
> Origin: http://example.com
> Sec-WebSocket-Key: sN9cRrP/n9NdMgdcy2VJFQ==
> Sec-WebSocket-Version: 13

- 与传统 HTTP 报文不同的地方

> Upgrade: websocket
> Connection: Upgrade

- 这两行表示发起的是 WebSocket 协议

> Sec-WebSocket-Key: sN9cRrP/n9NdMgdcy2VJFQ==
> Sec-WebSocket-Version: 13

- Sec-WebSocket-Key 是由浏览器随机生成的，提供基本的防护，防止恶意或者无意的连接。

- Sec-WebSocket-Version 表示 WebSocket 的版本，最初 WebSocket 协议太多，不同厂商都有自己的协议版本，不过现在已经定下来了。如果服务端不支持该版本，需要返回一个 Sec-WebSocket-Versionheader，里面包含服务端支持的版本号。

##### 服务端响应报文 ：

首先我们来看看服务端的响应报文：

> HTTP/1.1 101 Switching Protocols
> Upgrade: websocket
> Connection: Upgrade
> Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
> Sec-WebSocket-Protocol: chat

- 首先，101 状态码表示服务器已经理解了客户端的请求，并将通过 Upgrade 消息头通知客户端采用不同的协议来完成这个请求；
- 然后，Sec-WebSocket-Accept 这个则是经过服务器确认，并且加密过后的 Sec-WebSocket-Key；
- 最后，Sec-WebSocket-Protocol 则是表示最终使用的协议。

### 二. WebSocket编程

#### 1. WebSocket注解

**@ServerEndpoint 注解**

是一个类层次的注解，它的功能主要是将目前的类定义成一个 websocket 服务器端。注解的值将被用于监听用户连接的终端访问 URL 地址。

**@OnOpen 和 @OnClose 注解**

这两个注解的作用不言自明：他们定义了当一个新用户连接和断开的时候所调用的方法。

**@OnMessage 注解**

这个注解定义了当服务器接收到客户端发送的消息时所调用的方法。注意：这个方法可能包含一个 javax.websocket.Session 可选参数。如果有这个参数，容器将会把当前发送消息客户端的连接 Session 注入进去。

#### 2. 服务端WebSocket

```java
package com.epochong.chatroom.service;

import javax.websocket.*;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;

/**
 * @author epochong
 * @date 2019/8/3 11:54
 * @email epochong@163.com
 * @blog epochong.github.io
 * @describe 后端实现，ws协议
 * ServerEndpoint("/websocket")注解：把当前类标记为websocket类，ws协议的终端，和Servlet差不多
 */
@ServerEndpoint("/websocket")
public class WebSocket {
    /**
     * 绑定当前websocket会话
     * 每个浏览器窗口建立起来就是一个session会话
     */
    private Session session;

    /**
     * 当前端websocket建立连接就会调用这个方法
     * 添加session
     * @param session
     */
    @OnOpen
    public void onOpen(Session session) {
        //绑定当前session
        this.session = session;
    }

    @OnError
    public void onError(Throwable e) {
        System.out.println("WebSocket连接失败");
    }

    /**
     * 服务器收到了浏览器发送的信息
     * 群聊:{"msg":"777","type":1}
     * 私聊:{"to":"0-","msg":"33333","type":2}
     */
    @OnMessage
    public void onMessage(String msg) {
        
    }

    @OnClose
    public void onClose() {
       
    }
}
```

#### 3. 客户端WebSocket

websocket 连接到 websocket 服务器端。使用构造函数 new WebSocket()创建WebSocket实例管理后台数据，及逻辑处理，通过this.session.getBasicRemote().sendText(msg)，与页面的@OnMessage 实现信息传递。

**onOpen：**创建一个连接到服务器的连接时将会调用此方法。

**onError：** 当客户端 - 服务器通信发生错误时将会调用此方法。

**onMessage：**当从服务器接收到一个消息时将会调用此方法。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>测试WebSocket</title>
</head>
<body>
    请输入要发送的信息:
    <input type="text" id="text">
    <button onclick="sendMsg2Server()">发送信息</button>
    <hr><!--换行，有一个水平线-->
    收到服务端信息为:
    <div id="read_from_server"></div>
    <hr>
    <button onclick="closeWebSocket()">关闭websocket</button>

    <script>
        /*大部分浏览器都是支持websocket的除了老版本的IE浏览器所以要判断一下*/
        var websocket = null;
        /*window对象是浏览器对象*/
        if ('WebSocket' in window) {
            console.log("支持webcoket!");
            // 后端websocket地址
            websocket = new WebSocket("ws://localhost:8087/websocket?username=wangchong");
        }else {
            alert("浏览器不支持websocket!");
        }
        // 浏览器与服务端建立链接后回调方法
        websocket.onopen = function() {
            /*输出到控制台，日志，和弹窗差不多*/
            console.log("websocket连接成功");
        };
        // 建立websocket失败
        websocket.onerror = function() {
            console.log("websocket连接失败");
        };
        // 浏览器收到服务器信息回调这个方法，收到的信息包装在event中
        websocket.onmessage = function (event) {
            var msg = event.data;
            flushDiv(msg);
        };
        // websocket关闭
        websocket.onclose = function () {
            closeWebSocket();
        };
        // 窗口关闭（没有点击按钮直接关闭），主动将当前窗口websocket关闭
        window.onbeforeunload = function () {
            closeWebSocket();
        };
        // 将浏览器信息发送到服务端
        function sendMsg2Server() {
            /*获取输入框的内容*/
            var msg = document.getElementById("text").value;
            websocket.send(msg);
        }
        // 刷新当前div，收到服务器的信息刷新内容
        function flushDiv(msg) {
            document.getElementById("read_from_server").innerText = msg;
            print(msg);
        }
        function closeWebSocket() {
            console.log("关闭WebSocket")
            websocket.close();
        }
    </script>
</body>
</html>
```

## WebSocket 机制

### WebSocket的原理及运行机制

WebSocket是HTML5下一种新的协议。它实现了浏览器与服务器全双工通信，能更好的节省服务器资源和带宽并达到实时通讯的目的。它与HTTP一样通过已建立的TCP连接来传输数据，但是它和HTTP最大不同是：

WebSocket是一种双向通信协议。在建立连接后，WebSocket服务器端和客户端都能主动向对方发送或接收数据，就像Socket一样； WebSocket需要像TCP一样，先建立连接，连接成功后才能相互通信。

传统HTTP客户端与服务器请求响应模式如下图所示：

![image-20200318103954393](/Users/wangchong/Library/Application Support/typora-user-images/image-20200318103954393.png)

WebSocket模式客户端与服务器请求响应模式如下图：

![image-20200318104004955](/Users/wangchong/Library/Application Support/typora-user-images/image-20200318104004955.png)

上图对比可以看出，相对于传统HTTP每次请求-应答都需要客户端与服务端建立连接的模式，WebSocket是类似Socket的TCP长连接通讯模式。一旦WebSocket连接建立后，后续数据都以帧序列的形式传输。在客户端断开WebSocket连接或Server端中断连接前，不需要客户端和服务端重新发起连接请求。在海量并发及客户端与服务器交互负载流量大的情况下，极大的节省了网络带宽资源的消耗，有明显的性能优势，且客户端发送和接受消息是在同一个持久连接上发起，实时性优势明显。

### 相比HTTP长连接，WebSocket有以下特点：

是真正的全双工方式，建立连接后客户端与服务器端是完全平等的，可以互相主动请求。而HTTP长连接基于HTTP，是传统的客户端对服务器发起请求的模式。 HTTP长连接中，每次数据交换除了真正的数据部分外，服务器和客户端还要大量交换HTTP header，信息交换效率很低。Websocket协议通过第一个request建立了TCP连接之后，之后交换的数据都不需要发送 HTTP header就能交换数据，这显然和原有的HTTP协议有区别所以它需要对服务器和客户端都进行升级才能实现（主流浏览器都已支持HTML5）。此外还有 multiplexing、不同的URL可以复用同一个WebSocket连接等功能。这些都是HTTP长连接不能做到的。 下面再通过客户端和服务端交互的报文对比WebSocket通讯与传统HTTP的不同点：

在客户端，new WebSocket实例化一个新的WebSocket客户端对象，请求类似 ws://yourdomain:port/path 的服务端WebSocket URL，客户端WebSocket对象会自动解析并识别为WebSocket请求，并连接服务端端口，执行双方握手过程，客户端发送数据格式类似：

![image-20200318103847146](/Users/wangchong/Library/Application Support/typora-user-images/image-20200318103847146.png)

可以看到，客户端发起的WebSocket连接报文类似传统HTTP报文，Upgrade：websocket参数值表明这是WebSocket类型请求，Sec-WebSocket-Key是WebSocket客户端发送的一个 base64编码的密文，要求服务端必须返回一个对应加密的Sec-WebSocket-Accept应答，否则客户端会抛出Error during WebSocket handshake错误，并关闭连接。

服务端收到报文后返回的数据格式类似：

![image-20200318103859159](/Users/wangchong/Library/Application Support/typora-user-images/image-20200318103859159.png)

Protocols表示服务端接受WebSocket协议的客户端连接，经过这样的请求-响应处理后，两端的WebSocket连接握手成功, 后续就可以进行TCP通讯了。用户可以查阅WebSocket协议栈了解WebSocket客户端和服务端更详细的交互数据格式。

在开发方面，WebSocket API 也十分简单：只需要实例化 WebSocket，创建连接，然后服务端和客户端就可以相互发送和响应消息。在WebSocket 实现及案例分析部分可以看到详细的 WebSocket API 及代码实现。

## WebSocket优点

- 较少的控制开销。在连接创建后，服务器和客户端之间交换数据时，用于协议控制的数据包头部相对较小。在不包含扩展的情况下，对于服务器到客户端的内容，此头部大小只有2至10字节（和数据包长度有关）；对于客户端到服务器的内容，此头部还需要加上额外的4字节的掩码。相对于HTTP请求每次都要携带完整的头部，此项开销显著减少了。
- 更强的实时性。由于协议是全双工的，所以服务器可以随时主动给客户端下发数据。相对于HTTP请求需要等待客户端发起请求服务端才能响应，延迟明显更少；即使是和Comet等类似的长轮询比较，其也能在短时间内更多次地传递数据。
- 保持连接状态。于HTTP不同的是，Websocket需要先创建连接，这就使得其成为一种有状态的协议，之后通信时可以省略部分状态信息。而HTTP请求可能需要在每个请求都携带状态信息（如身份认证等）。
- 更好的二进制支持。Websocket定义了二进制帧，相对HTTP，可以更轻松地处理二进制内容。
- 可以支持扩展。Websocket定义了扩展，用户可以扩展协议、实现部分自定义的子协议。如部分浏览器支持压缩等。
- 更好的压缩效果。相对于HTTP压缩，Websocket在适当的扩展支持下，可以沿用之前内容的上下文，在传递类似的数据时，可以显著地提高压缩率。

WebSocket 是独立的、创建在 TCP 上的协议。

Websocket 通过 HTTP/1.1 协议的101状态码进行握手。

为了创建Websocket连接，需要通过浏览器发出请求，之后服务器进行回应，这个过程通常称为“握手”（handshaking）。

## websocket断线重连机制

在使用websocket的过程中，有时候会遇到网络断开的情况，但是在网络断开的时候服务器端并没有触发onclose的事件。这样会有：服务器会继续向客户端发送多余的链接，并且这些数据还会丢失。所以就需要一种机制来检测客户端和服务端是否处于正常的链接状态。因此就有了websocket的心跳了。还有心跳，说明还活着，没有心跳说明已经挂掉了。

一、心跳重连机制(考虑网络因素)
实现机制
心跳机制是每隔一段时间会向服务器发送一个数据包，告诉服务器自己还活着，同时客户端会确认服务器端是否还活着，如果还活着的话，就会回传一个数据包给客户端来确定服务器端也还活着，否则的话，有可能是网络断开连接了。需要重连~

代码实例（javascript）

```java
var WebSocket = {};
WebSocket.init = function(uri) {
    this.wsUri = uri;
    this.lastHeartBeat = new Date().getTime();
    this.overtime = 8000;
initChannelData();
WebSocket.websocket = new WebSocket(WebSocket.wsUri);

WebSocket.websocket.onopen = function(evt) {
    onOpen(evt)
};
WebSocket.websocket.onclose = function(evt) {
    onClose(evt)
};
WebSocket.websocket.onmessage = function(evt) {
    onMessage(evt)
};
WebSocket.websocket.onerror = function(evt) {
    onError(evt)
};
setInterval(checkConnect,5000);
}

function checkConnect() {
     doSend("{'event':'ping'}");
     if ((new Date().getTime() - WebSocket.lastHeartBeat) > WebSocket.overtime) {
         reConnect();
     }
 }
function onMessage(e) {
     WebSocket.lastHeartBeat = new Date().getTime();
}
/**

* 重新连接
* */
  function reConnect(){
    console.log("socket 连接断开，正在尝试重新建立连接");
    WebSocket.init("");
  }
```

二、onclose失去连接重连机制(不考虑网路因素)
实现机制
在onclose回调事件中再次连接

代码实例（javascript）

```java
var WebSocket = {};
WebSocket.init = function(uri) {
    this.wsUri = uri;
    this.lastHeartBeat = new Date().getTime();
    this.overtime = 8000;
initChannelData();
WebSocket.websocket = new WebSocket(WebSocket.wsUri);

WebSocket.websocket.onopen = function(evt) {
    onOpen(evt)
};
WebSocket.websocket.onclose = function(evt) {
    onClose(evt)
};
WebSocket.websocket.onmessage = function(evt) {
    onMessage(evt)
};
WebSocket.websocket.onerror = function(evt) {
    onError(evt)
};
}
/**

* 重新连接
* */
  function reConnect(){
    console.log("socket 连接断开，正在尝试重新建立连接");
    WebSocket.init("");
  }

function onClose(evt) {
    console.log("DISCONNECTED")
    reConnect();
}

```

## 应用场景

大部分传统的方式既浪费带宽（HTTP HEAD 是比较大的），又消耗服务器 CPU 占用（没有信息也要接受请求）；而 WebSocket 则会大幅降低上述的消耗，更适用于以下场景：

- 实时性要求高的应用
- 聊天室
- IoT (物联网 - internet of things)
- 在线多人游戏

多用户交互的 WebSocket 实例：

这里随便设想一个用户场景，比如我们要做一个在线纸牌游戏，肯定就是一个多人进入同一个房间的形式，并且每个人的动作能广播给其他人。