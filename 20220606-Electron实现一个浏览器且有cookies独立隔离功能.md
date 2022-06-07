# 20220606-Electron实现一个浏览器且有cookies独立隔离功能

## 最终效果图：

![image-20220606143904428](https://s2.loli.net/2022/06/06/r7tjqZN2cTx5D9m.png)

![image-20220606143918699](https://s2.loli.net/2022/06/06/oNQ4Lvk7IBaF2dj.png)

![image-20220606143942546](https://s2.loli.net/2022/06/06/cDAM5mkbELXWPT6.png)

## 功能需求清单

1. 实现一个和浏览器差不多的外观
2. 基本的功能要有，比如：前进、后退、刷新、新建标签、返回首页
3. 可以实现多账号同时登录，也就是每个打开的网页标签，可以登录不同的账号

## 需要开发的功能

实现上面的需求清单的功能，其实需要做的远不止上面的三点，我总结了实际需要实现的功能如下：

1. 首先是切图，先把基础的UI写出来：

   1.1 侧边栏：整个侧边栏做成可以收缩，里面的收藏夹、最近打开，可以收起/展开

   1.2 菜单栏基础功能的图标：前进、后退、刷新、首页、新建标签

   1.3 标签栏的新增、删除、切换

2. 网页的展示部分：

​		   2.1  网页的呈现与加载

​			2.2 标签栏的标题更新
​			2.3 网页加载状态的更新

​			2.4 网页内新链接打开的处理

3. 多账号同时登录的实现
   3.1 网页的cookies获取与本地保存
   3.2 加载网页时注入cookies
   3.3 加载网页时设置localStorage
4. 网页的调试功能: 打开控制台按钮

​			

## 功能的实现

### 一、侧边栏及菜单栏

- 侧边栏使用element-ui-plus的`Collapse`折叠面板组件
- 菜单栏自己手写一个

### 二、网页的展示

加载网页的方法有很多：

- browserWindow  

  > 创建并控制浏览器窗口，一般在主进程创建，全局只使用一个

- browserView  

  > `BrowserView`被用来让 `BrowserWindow` 嵌入更多的 web 内容。 它就像一个子窗口，除了它的位置是相对于父窗口。 这意味着可以替代webview标签。
  >
  > 但是它的缺点很明显：
  >
  > - 层级问题：新创建的`BrowserView`会覆盖在最顶部，而且这不是`z-index`能够控制的，会遮挡住其他内容
  > - 太过笨重：创建一个`BrowserView`相当于打开一个浏览器，加载的速度很慢，而且消耗内存

- iframe 

  > iframe加载网页确实很方便，但是它的能力也有限，只能提供基础的网页加载、通讯功能。很多需要自定义的功能无法实现。

- webview

  > webview相当于升级版的iframe，它轻量而且功能多，创建一个webview只需要写一个标签即可，而且没有`BrowserView`的层级问题，官方提供的api也很多，完全可以满足需求。

经过查看官方文档，结合以往的经验，选用electron的内置组件`webview`作为网页的载体容器。

```html
<div v-for="(item,index) in webviewList" :key="item.id" >
     <webview class="webview" :ref="`webview_${item.id}`" :partition="item.partition" :webPreferences="item.webpreferences"  :allowpopups="false" :src="item.url" :preload="preloadUrl"  nodeintegration autosize="on" v-show="activeTab==item.id"></webview>
   </div>
```

一个webview的属性包含：

```js
let id = dayjs().format('YYYYMMDDHHmmss')
let newWebview = {
          id:id,//唯一id
          name:'新标签页',//网页标签名称
          url:'https://www.baidu.com',//加载的地址
          loading:true,//加载状态
          partition:`persist:cookie_id_${id}`,
          webpreferences:``
        }
```

新建标签时，就是新增一个上面的对象，添加到`webviewList`数组里。

经过以上简单的代码，就可以成功加载网页了。

## 网页跳转的处理

webview加载出来的网页，你会发现里面的链接点了不会弹出新页面。其实这些都需要开发者自行处理的。

### 方法一：allowpopups

在标签上添加`allowpopups`属性后，代表允许弹出新标签，点击链接后会弹出一个新的子窗口，但是我这里的需求是需要在自己写的标签栏上新增一个标签，不符合需求。

### 方法二：自行拦截跳转

监听webview的`new-window`事件，当点击链接要跳转到新页面时就会触发该事件。此时我们可以提取里面的url参数。然后赋值给当前选中的webview，改变它的src路径，就能进行切换页面。

```js
webview.addEventListener('new-window', (e) => {
    let targrtWebview = targrt
    targrtWebview.loading = true
    let parser = new URL(e.url)
    const protocol = parser.protocol;
    if (protocol === 'http:' || protocol === 'https:') {
        this.$refs[`webview_${targrtWebview.id}`][0].src = e.url;
        targrtWebview.url = e.url
    }
})
```

效果：

![move2222](https://s2.loli.net/2022/06/07/8kuRBynlTiIVSQm.gif)

## 标签栏的网页标题

如上图实现的效果，切换网页后，按照正常的浏览器，会把网页的标题更新到tab栏，现在需要实现这个功能。

### page-title-updated事件

> [ page-title-updated (electronjs.org)](https://www.electronjs.org/zh/docs/latest/api/webview-tag#事件-page-title-updated)

这是webview提供的一个监听事件，当导航时页面标题更新时触发。

### webview.getTitle()

> [ Twebview.getTitle(electronjs.org)](https://www.electronjs.org/zh/docs/latest/api/webview-tag#webviewgettitle)

返回 `string` - 访客页的标题。

我们可以利用上面这两个api实现标题更新效果。

```js
 webview.addEventListener('page-title-updated', function () {
     targrt.name = webview.getTitle()		// 当title更新，更新tab栏的title
 });
```

## 标签栏的网页加载状态

观察上面的图片，在打开一个新的链接时，标签栏的左边会有一个loading转圈圈，直到网页加载完成后才消失。一般的浏览器也会有加载状态的显示，能够让用户知道网页的加载状态如何，这也是很有必要的。

### 新增tab时显示loading

简单的css动画实现转圈

```html
<div class="tab-loading" v-if="item.loading">
    <el-icon :size="12"><Loading /></el-icon>
</div>
```

新增tab的对象，设置loading为true，直到加载结束:`did-finish-load`触发才设置false

```js
let newWebview = {
          id:id,
          name:'新标签页',
          url:'https://www.baidu.com',
          loading:true,
          partition:`persist:cookie_id_${id}`,
          webpreferences:``
        }
```

```js
webview.addEventListener('did-finish-load',(e)=>{
    ...
    targrt.loading = false
})
```

### 切换链接时显示loading

当在一个网页里点击其他链接时，重新设置loading显示

```js
 webview.addEventListener('new-window', (e) => {
     let targrtWebview = targrt
     targrtWebview.loading = true
 })
```

## 前进/后退/刷新功能

![image-20220607111152997](https://s2.loli.net/2022/06/07/A4gQYLxKXRk3We6.png)

这三个功能可以使用时，会成激活颜色，不能使用时置灰无法点击。

- 前进：使用`Webview.canGoForward()`判断状态，`Webview.goForward()`调用前进
- 后退：使用`Webview.canGoBack()`判断状态，`Webview.goBack()`调用前进
- 刷新：使用`Webview.getURL()`判断有无加载链接，如果有就可以刷新，调用`Webview.reload()`重新加载网页

一开始想用`computed`自动计算状态，但是切换当前webview不会改变里面的属性，所以不能触发重新计算。所以就使用手动更新：

```js
 updateWebview() {
      if(this.webviewList.length== 0) {
        this.reset()
      }
      if(this.activeTab>0&&this.activeWebview) {
        this.getURL = this.activeWebview.getURL()
        this.canGoBack = this.activeWebview.canGoBack()
        this.canGoFordward = this.activeWebview.canGoForward()
        this.activeWebview = this.$refs[`webview_${this.activeTab}`][0]
      }
      else {
        this.getURL = ''
        this.canGoBack = false
        this.canGoFordward = false
      }
    }
```

每次切换或者点击前进，后退，刷新后，手动更新一下状态。

##  标签栏的新增、删除、切换

### 新增

写一个add方法，往`webviewList`这个数组里push新的对象，之后要监听新增的webview的各个事件，`webview.addEventListener`都写在这。

### 删除

删除看似只是调一下`splice()`就可以，但是这里有很多细节需要处理：

- 在当前激活的tab栏，删除别的tab
- 在当前激活的tab栏，删除当前激活的tab
- 如果tab只有一个，删除后的处理
- 如果tab大于两个，删除左边/右边/当前tab，之前显示哪个tab激活？

![move222222](C:/Users/PM/Desktop/gif/move222222.gif)

### 切换

切换使用生成的tab标签id作为唯一值

```
:class="{'active':activeTab==item.id}" //tab标签激活样式

v-show="activeTab==item.id" //webview设置v-show 
```

```js
let id = dayjs().format('YYYYMMDDHHmmss') + Math.random() * 100
```

## 多账号同时登录，cookies隔离的实现

### 实现的思路

不同标签登录不同账号，切换标签时不影响，做到这一点，我首先想到的是，不同的webview需要用不同的cookies，互相要做到独立，相互隔离。

翻遍了文档，终于找到了相关的介绍(详细介绍请看文档)

> [partition | Electron (electronjs.org)](https://www.electronjs.org/zh/docs/latest/api/webview-tag#partition)
>
> [session | Electron (electronjs.org)](https://www.electronjs.org/zh/docs/latest/api/session)
>
> [类：Cookies | Electron (electronjs.org)](https://www.electronjs.org/zh/docs/latest/api/cookie)

核心的整个流程大致为：

1. 在webview设置不同的`partition`
2. 使用session的`fromPartition`，通过传入不同的`partition`，得到不同的session实例
3. 使用`cookies.set`方法，给不同`partition`设置不同的cookies
4. 使用`electron-stroe`这个第三方库进行本地数据存储

这里我认为比较难的点在于其中的思路，对于不太熟悉electron的我，需要搜集大量的资料，结合自己的想法，尝试。代码量不多，重在前期的思路。方向对了，往下做就可以了。

### 网页的cookies获取与本地保存

渲染进程发送通知：

这里需要注意，事件的参数需要用`JSON.stringify`进行对象->字符串转换，接收时用`JSON.parse`解析，不然会报错。

```js
let targrt = this.webviewList.find(i=>i.id==id)
ipcRenderer.send('getCookies',JSON.stringify(targrt))
```

主进程接收：

```js
const {
    ipcMain,
    session 
} = require('electron')
const Store = require('electron-store');
const store = new Store();
ipcMain.on('getCookies',(event, target) => {
    try {
        target = JSON.parse(target)
        let {partition , url} = target || {}
        let targetUrl = new URL(url)
        let ses = session.fromPartition(partition);
        ses.cookies.get({url:targetUrl.origin})
            .then((cookies) => {
            if(cookies.length>0){
                cookies.forEach(i=>{
                    i['url'] = targetUrl.origin
                })
                store.set({
                    [partition]:cookies
                })
            }
            else {
                console.log("cookies为空");
            }
        }).catch((error) => {
            console.log(error)
        })
    }
    catch (err) {
        console.log("getCookies--err:",err);
    }
})
```

### 加载网页时注入cookies

渲染进程发送通知：

由于有时候设置了cookies，但是webciew的网页可能已经加载完成了，所以这时候需要刷新网页，让请求携带cookies才能返回正常登录状态。所以需要异步等待设置完cookies后，返回一个true结果用于判断。这里使用`ipcRenderer.invoke`。

```js
let targrt = this.webviewList.find(i=>i.id==id)
const result = await ipcRenderer.invoke('setCookies',{
    target:JSON.stringify(targrt),
    cookies:cookies
})
```

主进程响应：

```js
ipcMain.handle('setCookies',(event, args) => {
			try {
				let {target , cookies} = args || {}
				let {partition} = JSON.parse(target) || {} //获取唯一的标记值
				let ses = session.fromPartition(partition);
				cookies.forEach(details=>{
					ses.cookies.set(details)
				})
				return true
			}
			catch (err) {
				console.log("setCookies--err:",err);
			}
		})
```

到这里，已经成功给webview设置上cookies了

![image-20220607152937273](https://s2.loli.net/2022/06/07/fh4vnUo7bQOagjR.png)

### 注入localStorage

由于需要实现自动登录web端的项目等需求，顺手也做了注入localStorage的功能。

由于需要使用到`preload`预加载文件，需要设置`webpreferences:contextIsolation=false`。

渲染进程：

```js
webview.send('setToken') //设置登录token
webview.send('setProjectId',project_id) //设置全局项目id
```

webview的preload.js文件

需要注意preload.js的文件路径，需要自行拼接：

```js
preloadUrl:path.join(__static, 'html/client/preload.js')
```

```js
// 注入token
ipcRenderer.on('setToken',(e) => {
  // 设置token
  let token = store.get('Token') || ''
  let userInfo = store.get('userInfo') || ''
  localStorage.setItem('Token', JSON.stringify(token))
  localStorage.setItem('userInfo', JSON.stringify(userInfo))
})
//设置项目id
ipcRenderer.on('setProjectId',(event,id)=>{
  window.$vm.$bus.emit('setProjectId',id)
})
```

### electron-store

> [sindresorhus/electron-store: Simple data persistence for your Electron app or module - Save and load user preferences, app state, cache, etc (github.com)](https://github.com/sindresorhus/electron-store)

这个库可以在主进程和渲染进程使用。

```js
const Store = require('electron-store');
const store = new Store();
```

主进程设置:

```js
store.set({
	cookies:cookies
})
```

渲染进程获取:

```js
let cookies = store.get(item.cookies)
```

## 同类项目

> [cgq001/browser: 模仿一个PC浏览器 (github.com)](https://github.com/cgq001/browser)
>
> [cgq001/e-api-electron: E-API的Electron客户端 (github.com)](https://github.com/cgq001/e-api-electron)
>
> [E-API官网 (nodebook.top)](http://e-api.nodebook.top/#/deploy)
