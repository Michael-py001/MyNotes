# 20220418-Electron中对webview实现keep-alive和懒加载

> 参考文章：
>
> [Vue中对iframe实现keep alive_墨工的博客-CSDN博客_vue实现iframe效果](https://blog.csdn.net/u012831062/article/details/103972070)

## 懒加载webview

因为webview和iframe一样，不能作为虚拟节点，所以

APP.vue代码

```html
<template>

	<electronMenu ref="electronMenu" @init="electronMenuInit" :loading="loading"/>
	<div class="app-content u-scrollbar">
		<router-view v-slot="{ Component }">
		<keep-alive v-if="$route.meta.keepAlive">
			<component :is="Component" />
		</keep-alive>
		<component :is="Component" v-else />
	</router-view>
	<my-webview 
		v-model:loading="loading" 
		:info="item" 
		@saveWebviewId="saveWebviewId" 
		v-show="$route.path==item.path" 
		:hasSecondTab="item.showElectronTabCard" 
		v-for="(item,index) in allOpenWebview" 
		:key="item.name"
	/>
	</div>
</template>
```

`electronMenu`是左侧固定的侧边栏

`router-view`是中间的动态渲染的组件，用`keep-alive`包裹

`my-webview `是动态渲染的右侧webview部分

### 收集路由表信息

> 收集好路由表信息，加上`hasOpen`标记，默认为`false`，加载过的路由就设置成`true`

```js
// 收集路由
collectRoutes() {
    let webviewRoutes = []
    this.$router.options.routes.forEach(item => {
        if (item.hasWebview) {
            webviewRoutes.push({
                ...item,
                hasOpen:false
            })
        }
    })
    this.SaveWebviewRoutes(webviewRoutes)
    return webviewRoutes
},
```

### 设置当前加载路由标记

```js
//根据当前路由设置hasOpen
setWebviewOpen(route) {
    let target = this.webviewRoutes.find(i => i.path == route.path)
    if(target && !target.hasOpen) {
        target.hasOpen = true
    }
},
```

### 动态计算已打开过的路由

```js
computed: {
    allOpenWebview() { //打开过的webview
        return this.webviewRoutes.filter(i=>i.hasOpen)
    }
},
```

`my-webview`标签会遍历`allOpenWebview`这个数组，这里都`hasOpen`为`true`的路由

### 显示当前路由的webview

`my-webview`的 `v-show="$route.path==item.path"`，只有当前路由路径和`allOpenWebview`里面记录的路由元素相同才显示。



## my-webview组件的内容

```html
<template>
  <div class="ViewForOverview u-scrollbar" :class="{'hasSecondTab':hasSecondTab}">
    <div class="loading" v-if="loading">
      <div class="el-loading-spinner"><svg class="circular" viewBox="25 25 50 50"><circle class="path" cx="50" cy="50" r="20" fill="none"></circle></svg><!----></div>
    </div>
    <div class="open">
      <el-button class="open" size="small" @click="openDev">DevTool</el-button>
    </div>
    <webview ref="ViewForOverview" class="u-scrollbar" style="display:inline-flex;" id="webview" :preload="pathUrl" disablewebsecurity nodeintegration autosize="on" :webpreferences="webpreferences"></webview>
  </div>
</template>
```

主要有三部分：

1. loading，加载未完成时显示
2. 打开控制台按钮`DevTool`
3. `webview`

### webview的重要属性

1. `preload` 会加载访客页面时，最优先加载一个js的文件，优先级是最高的，会在页面所有js加载之前就加载，也比vue所有生命周期早。这里面可以使用node的功能，可以和渲染进程通讯。

2. `nodeintegration` 开启node功能，如果不开启，`preload`里不能使用Node

3. `webpreferences` 这个内容和创建`browserWindow`的参数一样，不一样的是传入的格式内容不能是对象，而是字符串，属性和内容之间用`=`连接。

   ```js
   webpreferences:'contextIsolation=false',
   ```

   这里传入了一个很重要的属性，关闭环境隔离，如果不关闭，在`preload`里获取的window和访客页的window是相互独立的，挂载的属性不能相互访问，如果要在preload里使用访客页window里挂载的一些方法，比如路由跳转，这时关闭即可。(这里一开始不知道有这回事，卡住了很久，翻了很久官方的英文文档才知道)

## webview加载网页的方式

1. 在`mounted`生命周期里，给webview设置src属性：webview.setAttribute('src', 'http://xxx')

   ```js
   const webview = this.$refs.ViewForOverview
   this.webviewContent = webview
   webview.setAttribute('src',this.currentUrl)
   ```

   

2. 直接在标签上设置src属性

   ```html
   <webview :src="currentUrl" id="webview"></webview>
   ```

   `currentUrl`在`comouted`里自动计算就好了

## 渲染进程与webview访客页通讯

渲染进程与访客页通讯依靠的是`webview`的`id`，所以在webview加载完成后，就需要把它的id保存起来。因为创建的webview不只一个，所以后续根据这个id与指定的webview通讯。

```js

//mounted里面
const webview = this.$refs.ViewForOverview
let that = this
webview.addEventListener("dom-ready", function() {
      let info = {
        id:webview.getWebContentsId(),
        path:that.$route.path,
        name:that.$route.name.replace('.Home','')
      }
      that.$emit('saveWebviewId',{
       ...info
      })
      that.$route['webviewInfo'] = info //保存webview ID
      //id存储起来,后续指定webview进行通信
      ipcRenderer.send('getWebviewId', {webviewInfo:info})
    });
```

vuex state里的数据，记录每个页面的webview id 和包含的页面路径

```js
//vuex state里的数据
totalRouteWebview:{ //页面容器
		'ResourceLibrary':{
			webviewId:0,
			pages:[
				'ResourceLibrary.Image',
				'ResourceLibrary.Video',
				'ResourceLibrary.TitlePack',
				'ResourceLibrary.OrientationPackage',
				'ResourceLibrary.LabelBag',
			]
		},
		'Manage':{
			webviewId:0,
			pages:[
				'Manage.AccountManagement',
				'Manage.ProjectManagement'
			]
		},
		'Report':{
			webviewId:0,
			pages:[
				'Overview.Home'
			]
		},

	}
```

思路是在路由表里Vuex里记录一份，在主进程里也记录一份，最后用的时候根据页面，在vuex里找到对应的id，再和主进程保存的id进行对比，如果一致，则在主进程里用`webContents.fromId(id).send()`与这个id的webview通讯。

```js
ipcMain.on('getWebviewId',(event, arg) => {
    let {name,path,id} = arg.webviewInfo
    let target = this.allWebviewList.find(i => i.name == name) 
    if(target) {
        console.log("已经存在:",arg.webviewInfo);
        target.id = id
    }
    else {
        this.allWebviewList.push(arg.webviewInfo) //保存创建了的webview 
    }
    event.returnValue = true
})
```

## 渲染进程与主进程通讯

当点击侧边栏时，与主进程进行通讯。

```js
// App.vue
this.$bus.on('clickSecondTab', (data) => {
    let {parentRoute,targetRoute} = data
    let webviewId = this.totalRouteWebview[parentRoute]['webviewId']
    ipcRenderer.send('changeRoute',{
        webviewId,
        targetRoute
    })
})
```

```js
//主进程	
const {
     ipcMain,
     webContents
    } = require('electron')
    // 切换webview页面的路由
ipcMain.on('changeRoute', (event, arg) => {
    let {webviewId,targetRoute} = arg
    let target = this.allWebviewList.find(i => i.id === webviewId)
    //从以打开的路由记录寻找对应id的项
    if (target) {
        //发送
        webContents.fromId(target.id).send('client-changeRoute', targetRoute)
    }
})
```

## 主进程与webview通讯

主要用到的api：`webContents.fromId(id).send('eventName',data)`

- 主进程上面的代码：给webview发送事件

```js
webContents.fromId(target.id).send('client-changeRoute', targetRoute)
```

- webview中的`preload.js`中监听事件

```js
window.ipcRenderer.on("client-changeRoute",(event, arg) => {
    //传过来的是路由地址，进行跳转
  window.$vm.$nav(arg.toString())//window挂载的app实例
})
```



## webview.loadURL()的使用

实际情况中，使用loadURL()这个方法，webview标签必须先设置src属性，不然`dom-ready`没有触发前，webview的方法都是不能使用的。

## 常见报错



Error invoking remote method ‘ELECTRON_GUEST_VIEW_MANAGER_CALL‘: Error: (-3) loading

> [Error invoking remote method ‘ELECTRON_GUEST_VIEW_MANAGER_CALL‘: Error: (-3) loading_Stud_movingj的博客-CSDN博客](https://blog.csdn.net/SDFGS54DGF5S4/article/details/115339899)

相关issue可参考

> [Error: Error invoking remote method 'ELECTRON_GUEST_VIEW_MANAGER_CALL': Error: (-3) loading 'https://www.baidu.com/' · Issue #24171 · electron/electron (github.com)](https://github.com/electron/electron/issues/24171)
>
> [Twitter does not load inside Webview · Issue #25421 · electron/electron (github.com)](https://github.com/electron/electron/issues/25421)
>
> [[Bug\]: BrowserWindow#loadURL() throw an error when I load same page with different location hash · Issue #28208 · electron/electron (github.com)](https://github.com/electron/electron/issues/28208)
>
> [[Bug\]: Error: Error invoking remote method 'GUEST_VIEW_MANAGER_CALL': Error: ERR_FAILED (-2) · Issue #29188 · electron/electron (github.com)](https://github.com/electron/electron/issues/29188)
>
> [Electron 9.1.0 Webview throws error when setting src attribute · Issue #24608 · electron/electron (github.com)](https://github.com/electron/electron/issues/24608)
>
> [Error: Error invoking remote method 'ELECTRON_GUEST_VIEW_MANAGER_CALL': Error: (-3) loading 'https://www.baidu.com/' · Issue #24171 · electron/electron (github.com)](https://github.com/electron/electron/issues/24171)
>
> [Segfault in content::RenderFrameHostManager::SetRWHViewForInnerContents when navigating webview · Issue #25469 · electron/electron (github.com)](https://github.com/electron/electron/issues/25469)
>
> 



## 相关webview文章

> [electron代码，browserWindow的preload.js作用范围是哪里？ (newsn.net)](https://newsn.net/say/electron-browserwindow-preloadjs.html#electron@12开始的注意事项)



## app.getAppPath() 为项目目录下的`dist_electron`的绝对路径