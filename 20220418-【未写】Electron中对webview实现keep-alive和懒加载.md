# 20220418-【未写】Electron中对webview实现keep-alive和懒加载

> 参考文章：
>
> [Vue中对iframe实现keep alive_墨工的博客-CSDN博客_vue实现iframe效果](https://blog.csdn.net/u012831062/article/details/103972070)

## 懒加载webview



## webview加载网页的方式

1. webview.setAttribute('src', 'http://xxx')
2. 直接在标签上设置src属性

## 渲染进程与访客页通讯



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
>
> 



## app.getAppPath() 为项目目录下的`dist_electron`的绝对路径