# HTML5全屏显示API

摘自MDN: https://developer.mozilla.org/zh-CN/docs/Web/API/Fullscreen_API

 **`全屏 API`** 为使用用户的整个屏幕展现网络内容提供了一种简单的方式，并且在不需要时退出全屏模式。这种 API 让你可以简单地控制浏览器，使得一个元素与其子元素，如果存在的话，可以占据整个屏幕，并在此期间，从屏幕上隐藏所有的浏览器用户界面以及其他应用。

可以在[全屏 API 指南](https://developer.mozilla.org/zh-CN/docs/Web/API/Fullscreen_API/Guide)这篇文章了解更多细节。

## 进入全屏

> 全屏显示 需要指定任意一个Element标签，这里的document.documentElement 表示整个文档流全屏，当然也可以指定一个div全屏

用法：`Element.requestFullscreen()`，不同的浏览器有不同的前缀。

```js
 var el = document.documentElement; 
//全屏显示 需要指定任意一个Element标签，这里的document.documentElement 表示整个文档流全屏，当然也可以指定一个div全屏
        var rfs =
          el.requestFullScreen ||
          el.webkitRequestFullScreen ||
          el.mozRequestFullScreen ||
          el.msRequestFullscreen;
        if (typeof rfs != "undefined" && rfs) {
          rfs.call(el);
        }
```

### Element.requestFullscreen()是一个Promise

`Element.requestFullscreen()` 方法用于发出异步请求使元素进入全屏模式。

​		调用此API并不能保证元素一定能够进入全屏模式。如果元素被允许进入全屏幕模式，返回的[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)会resolve，并且该元素会收到一个`fullscreenchange (en-US)`事件，通知它已经进入全屏模式。如果全屏请求被拒绝，返回的promise会变成rejected并且该元素会收到一个`fullscreenerror (en-US)`事件。如果该元素已经从原来的文档中分离，那么该文档将会收到这些事件。

​		早期的Fullscreen API实现总是会把这些事件发送给document，而不是调用的元素，所以你自己可能需要处理这样的情况。参考 [Browser compatibility](https://developer.mozilla.org/zh-CN/docs/Web/Events/fullscreen#Browser_compatibility) in [[Page not yet written\]](https://developer.mozilla.org/zh-CN/docs/Web/Events/fullscreen) 来得知哪些浏览器做了这个改动。

### 语法：

```js
var Promise = Element.requestFullscreen(options);
```

### [参数](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/requestFullScreen#参数)

`options` 可选

一个[`FullscreenOptions`](https://developer.mozilla.org/zh-CN/docs/Web/API/FullscreenOptions)对象提供切换到全屏模式的控制选项。目前，唯一的选项是[`navigationUI` (en-US)](https://developer.mozilla.org/en-US/docs/Web/API/FullscreenOptions/navigationUI)，这控制了是否在元素处于全屏模式时显示导航条UI。默认值是`"auto"`，表明这将由浏览器来决定是否显示导航条。

[返回值](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/requestFullScreen#返回值)

在完成切换全屏模式后，返回一个已经用值 `undefined` resolved的[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)

[异常](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/requestFullScreen#异常)

*`requestFullscreen()`* *通过拒绝返回的 `Promise`来生成错误条件，而不是抛出一个传统的异常。拒绝控制器接收以下的某一个值：*

- [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError)

  在以下几种情况下，会抛出`TypeError`：

- 文档中包含的元素未完全激活，也就是说不是当前活动的元素。
- 元素不在文档之内。
- 因为功能策略限制配置或其他访问控制，元素不被允许使用`"fullscreen"`功能。
- 元素和它的文档是同一个节点。 



### [示例](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/requestFullScreen#示例)

在调用`requestFullScreen()之前，可以对``fullscreenchange (en-US)` 和 `fullscreenerror (en-US)`事件进行监听，这样在请求进入全屏模式成功或者失败时都能收到事件通知。

## 退出全屏

> 退出全屏必须用到`document`

用法：`Document.exitFullscreen()`，不同的浏览器有不同的前缀。

```js
 if (document.exitFullscreen) {
          //退出全屏  需要document来执行，不能是Element标签
          document.exitFullscreen();
        } else if (document.mozCancelFullScreen) {
          document.mozCancelFullScreen();
        } else if (document.webkitCancelFullScreen) {
          document.webkitCancelFullScreen();
        } else if (document.msExitFullscreen) {
          document.msExitFullscreen();
        }
```

## iframe标签进入全屏

iframe标签需要设置`allowfullscreen`属性才能进入全屏.

设置为`true`时，可以通过调用 `<iframe>` 的 [`requestFullscreen()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/requestFullScreen) 方法激活全屏模式。

这是一个历史遗留属性，已经被重新定义为 `allow="fullscreen"`。