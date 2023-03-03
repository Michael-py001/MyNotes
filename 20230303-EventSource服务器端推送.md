# 20230303-EventSource服务器端推送

> [EventSource - Web API 接口参考 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/API/EventSource)
>
> [使用服务器发送事件 - Web API 接口参考 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/API/Server-sent_events/Using_server-sent_events)
>
> [【JavaScript】论一个低配版Web实时通信库是如何实现的之二（ EventSource篇) - 外婆的 - 博客园 (cnblogs.com)](https://www.cnblogs.com/penghuwan/p/11391705.html)
>
> [【Node.js】论一个低配版Web实时通信库是如何实现的1（ WebSocket篇) - 外婆的 - 博客园 (cnblogs.com)](https://www.cnblogs.com/penghuwan/p/11381182.html#_label1)

EventSource是一个HTML5新增的JavaScript API，它允许Web浏览器从服务器接收推送事件。使用EventSource，我们可以通过HTTP连接实时获取服务器发出的事件，而无需像传统的HTTP请求那样在每次请求之间进行轮询。

EventSource是一个与服务器持久性连接的单向通信通道。当连接建立时，服务器可以向客户端发送事件流，而客户端可以在事件到达时进行处理。

一个 EventSource 实例会对 HTTP 服务开启一个持久化的连接，以`text/event-stream` 格式发送事件，会一直保持开启直到被要求关闭。

一旦连接开启，来自服务端传入的消息会以事件的形式分发至你代码中。如果接收消息中有一个事件字段，触发的事件与事件字段的值相同。如果没有事件字段存在，则将触发通用事件。

与 [WebSockets](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSockets_API),不同的是，服务端推送是单向的。数据信息被单向从服务端到客户端分发。当不需要以消息形式将数据从客户端发送到服务器时，这使它们成为绝佳的选择。例如，对于处理社交媒体状态更新，新闻提要或将数据传递到客户端存储机制（如 IndexedDB 或 Web 存储）之类的，EventSource 无疑是一个有效方案。

### 属性

#### readyState

- `CONNECTING`：表示正在与服务器端建立连接。
- `OPEN`：表示连接已经建立，客户端正在接收服务器端推送的消息数据。
- `CLOSED`：表示连接已经关闭，客户端无法接收推送的消息数据。

#### url

- 表示服务器端发送事件的地址。

#### withCredentials

- 表示是否允许在发送跨域请求时携带凭证。
- 如果设置为 `true`，则可以发送包括 Cookie 和验证信息在内的跨域请求的凭证，否则不允许。

其作为EventSource()的第二个参数来设置 withCredentials 属性。

EventSource 构造函数的语法如下：

```js
var eventSource = new EventSource(url[, eventSourceInitDict]);
```

其中，`url` 参数表示服务器端发送事件的地址；`eventSourceInitDict` 参数是一个可选参数，用于设置一组可选的参数，其中包含 withCredentials 属性。

例如，下面的示例代码展示了如何创建一个设置了 withCredentials 属性的 EventSource 对象：

```js
var eventSource = new EventSource("http://example.com/events", { withCredentials: true });
```

在上面的示例中，第二个参数 `{ withCredentials: true }` 表示设置了 withCredentials 属性为 `true`，允许发送跨域请求中的凭证。

### 方法

#### close()

- 用来关闭当前连接。

#### addEventListener()

- 用来为指定事件类型添加事件处理程序。

- 接收两个参数：事件类型和事件处理程序函数。

- 事件类型有三种：`open` 表示连接成功时触发，`message` 表示收到服务器端推送的消息时触发，`error` 表示连接发生错误时触发。

  ```js
  eventSource.addEventListener('message', function(event){
      console.log(event); // 打印完整的事件对象
  });
  ```

  

#### removeEventListener()

- 用来移除指定事件类型的事件处理程序。

### DOM 事件

还有一些 DOM 事件可用于处理EventSource对象，例如：

#### onopen

- 表示连接成功时触发此事件。
- 此事件不需要手动添加事件处理程序，直接写在 EventSource 对象上即可，如：source.onopen = function() { ... };

#### onmessage

- 表示收到服务器端推送的消息时触发此事件。
- 此事件不需要手动添加事件处理程序，直接写在 EventSource 对象上即可，如：source.onmessage = function(event) { ... };

#### onerror

- 表示连接发生错误时触发此事件。
- 此事件不需要手动添加事件处理程序，直接写在 EventSource 对象上即可，如：source.onerror = function(event) { ... };

## 案例一

> [dom-examples/index.html at main · mdn/dom-examples (github.com)](https://github.com/mdn/dom-examples/blob/main/server-sent-events/index.html)

## 案例二

```html
<!DOCTYPE html>
<html>
<head>
	<title>EventSource Demo</title>
	<script type="text/javascript">
		var eventSource = new EventSource('server-side.php');
		
		eventSource.onmessage = function(event) {
		   console.log(event);
		   var data = JSON.parse(event.data);
		   document.getElementById('result').innerHTML = data.message;
		};
	</script>
</head>
<body>
	<h1>EventSource Demo</h1>
	<div id="result"></div>
</body>
</html>
```

一旦连接建立，服务器可以使用“发送事件”命令将事件发送到客户端。这可以通过以下代码完成：

```php
<?php
header('Content-Type: text/event-stream');
header('Cache-Control: no-cache');

$data = array('message' => 'Hello World!');
echo 'data: ' . json_encode($data) . "\n\n";
flush();
?>
```

​	服务器端代码使用`header()`函数设置`Content-Type`和`Cache-Control`响应头，以确保数据以text/event-stream格式发送。在这个例子中，服务器端PHP代码使用`echo`输出一个JSON字符串，代表一个事件，客户端通过EventSource API能够捕获到这个事件并显示其中的数据。

本例子演示了如何使用EventSource API和服务器端脚本在Web应用程序中实现实时通信。在实际情况中，可以根据需要对服务器端脚本进行扩展，并使用EventSource API捕获和处理更多的推送事件。

​	客户端收到事件后，可以处理事件并根据需要执行操作。由于EventSource是单向通信，因此不支持从客户端向服务器发送任何消息。

## EventSource和text/event-stream有什么联系？

EventSource是利用text/event-stream格式在服务器和客户端之间建立流式连接的API，它是HTML5新增的API之一，用于实现服务器和客户端之间的实时通信。

text/event-stream是一种通过HTTP传输流的数据格式，它用于在客户端和服务器之间建立持久的单向连接，以实现推送消息的功能。它是适用于服务端推送事件的最简单的实现方式之一。

EventSource API使用text/event-stream格式来传输Web服务器中的事件，通过这种格式可以实现服务器向客户端推送数据，客户端每收到一条数据，都会触发一次事件，从而实现实时通信。

在使用EventSource时，需要服务器支持text/event-stream格式，以便EventSource可以通过这种格式与服务器建立连接并接收推送的事件流。

## EventSource和XMLHttpRequest

在使用EventSource API时，不需要使用XMLHttpRequest对象来发送请求，因为EventSource API已经通过`new EventSource()`内置实现了基于text/event-stream的持久HTTP连接。在客户端的JavaScript中，可以使用以下代码来建立EventSource的连接：

```js
var eventSource = new EventSource('server-side.php');
eventSource.onmessage = function(event) {
    console.log(event.data);
}
```

​	在这个示例中，客户端JavaScript代码通过`new EventSource()`来建立一个到`server-side.php`页面的连接。Server-side.php是返回text/event-stream格式数据的一个服务器端文件。客户端使用`onmessage`事件监听EventSource收到的消息，并在控制台打印出消息内容。

​	因此，使用EventSource API时，我们不需要自行设置text/event-stream请求头，EventSource API会自动处理连接并接收推送过来的事件数据。同时，服务器端也需要支持text/event-stream格式，以便EventSource API能够收到正确的事件数据。