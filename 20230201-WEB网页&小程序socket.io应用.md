# 20230201-WEB网页 & 小程序socket.io应用

## weapp.socket.io 

> [weapp-socketio/weapp.socket.io: A WebSocket client for building WeChat Mini Program implement by socket.io (github.com)](https://github.com/weapp-socketio/weapp.socket.io#readme)

![image-20230201163625602](https://s2.loli.net/2023/02/01/8ykwWMcpzoPhRDb.png)

### 连接&监听

- `socket.onconnect()` 监听连接是否成功
- `socket.on()` 监听某个事件，比如：`disconnect` 、`auction`

```js
import io from 'weapp.socket.io';
socket = io(BASE_URL + '?projectId=' + this.data.options.assetProjectId,{
      transports:['websocket'],
      reconnectionDelay:5,
      forceNew:true
    });
    socket.onconnect(() => {
      // console.log('连接成功')
    });
    socket.on('disconnect', data => {
      // console.log('断开成功',data)
    });
    socket.on('auction', data => {
     console.log('监听数据',data)
    });
```

### 断开连接

`socket.disconnect()`

```js
 closeSocket(){
    if(socket) {
      socket.disconnect();
    }
  }
onUnload: function(){
    this.closeSocket();
},

    onHide: function() {
        this.closeSocket();
    },
```

## socket.io

> [介绍 | Socket.IO](https://socket.io/zh-CN/docs/v4/)

### 什么是 Socket.IO[](https://socket.io/zh-CN/docs/v4/#what-socketio-is)

Socket.IO 是一个库，可以在客户端和服务器之间实现 **低延迟**, **双向** 和 **基于事件的** 通信。

![Diagram of a communication between a server and a client](https://s2.loli.net/2023/02/02/JFGd3kNljbfsR48.png)

它建立在 [WebSocket](https://fr.wikipedia.org/wiki/WebSocket) 协议之上，并提供额外的保证，例如回退到 HTTP 长轮询或自动重新连接。

### 示例

*服务器*

```js
import { Server } from "socket.io";

const io = new Server(3000);

io.on("connection", (socket) => {
  // 向客户端发送消息
  socket.emit("hello from server", 1, "2", { 3: Buffer.from([4]) });

  // 从客户端接收消息
  socket.on("hello from server", (...args) => {
    // ...
  });
});
```

### 客户端

```js
import { io } from "socket.io-client";

const socket = io("ws://localhost:3000");

// 向服务器发送消息
socket.emit("hello from server", 5, "6", { 7: Uint8Array.from([8]) });

// 从服务器接收消息
socket.on("hello from server", (...args) => {
  // ...
});
```

### io([url]\[, options])

- url  接口地址 (默认是 `window.location`)
- options 参数 
  - forceNew  (whether to create a new connection)
  - 更多参数请看：[客户端配置 | Socket.IO](https://socket.io/zh-CN/docs/v4/client-options/#low-level-engine-options)
- Returns [`<Socket>`](https://socket.io/zh-CN/docs/v4/client-api/#socket)

### 示例

```js
import { io } from "socket.io-client";

const socket = io("ws://example.com/my-namespace", {
  reconnectionDelayMax: 10000,
  auth: {
    token: "123"
  },
  query: {
    "my-key": "my-value"
  }
});
```

简短版

```js
import { Manager } from "socket.io-client";

const manager = new Manager("ws://example.com", {
  reconnectionDelayMax: 10000,
  query: {
    "my-key": "my-value"
  }
});

const socket = manager.socket("/my-namespace", {
  auth: {
    token: "123"
  }
});
```

## PC网页中的使用

> [balderdashy/sails.io.js: Browser SDK for communicating w/ Sails via sockets (github.com)](https://github.com/balderdashy/sails.io.js)

![image-20230214135240206](https://s2.loli.net/2023/02/14/KUbRrt1ATf5mPXI.png)

![image-20230214104437187](https://s2.loli.net/2023/02/14/KXeB4taubGpf53A.png)

![image-20230214104520478](https://s2.loli.net/2023/02/14/jtePi9aqANsZ3Ey.png)

![image-20230214105003045](https://s2.loli.net/2023/02/14/9lpzRGUWBbw7OCM.png)

## 关于为啥可以直接用io.connect而不是io.sails.connect()

> [Socket client (sailsjs.com)](https://sailsjs.com/documentation/reference/web-sockets/socket-client)

## CORS相关

[CORS (sailsjs.com)](https://sailsjs.com/documentation/concepts/security/cors)
