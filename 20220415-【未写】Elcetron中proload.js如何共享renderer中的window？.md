# 20220415-Elcetron中proload.js如何共享renderer中的window？

> 重点文档
>
> [webview-tag webpreferences](https://www.electronjs.org/zh/docs/latest/api/webview-tag#webpreferences)
>
> [BrowserWindow | Electron (electronjs.org)](https://www.electronjs.org/zh/docs/latest/api/browser-window#new-browserwindowoptions)
>
> [Context Isolation | Electron (electronjs.org)](https://www.electronjs.org/zh/docs/latest/tutorial/context-isolation#before-context-isolation-disabled)

我在Electron里用webview标签内嵌了一个另外项目的web页面，这个网页的window是挂载了`$vm`等全局变量的。

我目前的需求是：在客户端点击菜单，web端跳转到对应的页面，这里需要用到全局挂载的路由跳转方法。

- web端的main.js

```js
//web端的main.js
const $vm = app.use(store).use(router).mount('#app')
window.$vm = $vm
```

- 客户端的preload.js

  ```js
  window.aaa = "123"
  console.log('window.$vm',window.$vm)
  ```

讲道理这不都是同一个window吗，应该在preload.js里面输出的window跟web端的window是同一个东西，所以挂载上去的$vm应该能访问到才对。

但是事实上并不是我想的这样，这两个地方输出的window不是同一个东西，在preload中的window输出只有在proload挂上去的属性，在web端挂在window上的属性一个都没有。这可以判断出这两个环境是被隔离的，不是同一个环境。

图一：preload输出的window

![image-20220415145003539](https://s2.loli.net/2022/04/15/aQg823nIBx5LXCY.png)

图二：浏览器控制台输出的window

![image-20220415145455262](https://s2.loli.net/2022/04/15/9qVs7TfCdY5tGDm.png)

## contextIsolation 环境隔离

经过网上一番查询，基本没有答案，但是搜到了一个关键词:`contextIsolation`，我的理解是`环境隔离`的意思。在官网上的文档介绍：

![image-20220415145836317](https://s2.loli.net/2022/04/15/PEoQ4A3rKZxksFB.png)

翻译过来就是说：这个选项默认是true，每一个preload里面的代码，都运行在一个独立的文档和全局的window,还有他自己的JavaScript 内置方法和属性。这些在访客页面都是不可见的，这样做是为了保护不被访客页执行preload里的代码和调用electron的API（省略...）。

那么是不是在webview设置这个为false就可以共享一个环境了呢？

翻了下文档，发现webview标签也有`webpreferences`这个属性，

### [webpreferences](https://www.electronjs.org/zh/docs/latest/api/webview-tag#webpreferences)

```html
<webview src="https://github.com" webpreferences="allowRunningInsecureContent, javascript=no"></webview>
```

A `string` which is a comma separated list of strings which specifies the web preferences to be set on the webview. The full list of supported preference strings can be found in [BrowserWindow](https://www.electronjs.org/zh/docs/latest/api/browser-window#new-browserwindowoptions).

The string follows the same format as the features string in `window.open`. A name by itself is given a `true` boolean value. A preference can be set to another value by including an `=`, followed by the value. Special values `yes` and `1` are interpreted as `true`, while `no` and `0` are interpreted as `false`.

这里需要注意，这个`webpreferences`格式是字符串，不是对象，true/false是用`=`来连接

所以最后给`webview`标签设置了`webpreferences:'contextIsolation=false'`，这下子终于搞定了这个难题