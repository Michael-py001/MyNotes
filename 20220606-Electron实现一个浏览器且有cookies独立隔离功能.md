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

监听webview的`new-window`事件，当点击链接要跳转到新页面时就会触发该事件。此时我们可以提取里面的url参数。

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

