# 20220411-Electron主进程/渲染进程与webview通讯

参考文章

> [electron 内主进程与渲染进程，webview之间的通信 - 苏氏之道 - 博客园 (cnblogs.com)](https://www.cnblogs.com/suzhen-2012/p/9932662.html)
>
> [electron通信 主进程与渲染进程/主进程与webview通信 | 独书先生 (dushusir.com)](http://dushusir.com/electron-ipcmain-ipcrenderer/)
>
> [研究Electron主进程、渲染进程、webview之间的通讯 - 海角在眼前 - 博客园 (cnblogs.com)](https://www.cnblogs.com/lovesong/p/11180336.html)
>
> [electron-playground文档 · 语雀 (yuque.com)](https://www.yuque.com/ezg6c4/op375w)
>
> [electron发送消息给webview里的子iframe - (dagoogle.cn)](http://www.dagoogle.cn/n/1133.html)
>
> [electron程序，表象分析iframe和webview嵌入页面的表现 (newsn.net)](https://newsn.net/say/electron-iframe-vs-webview-xframe.html)
>
> [electron代码，browserWindow的preload.js作用范围是哪里？ (newsn.net)](https://newsn.net/say/electron-browserwindow-preloadjs.html)
>
> [electron webview加载远程preload方法 - SegmentFault 思否](https://segmentfault.com/a/1190000022484086?utm_source=sf-similar-article)
>
> [electron发送消息给webview里的子iframe - SegmentFault 思否](https://segmentfault.com/a/1190000022534874?utm_source=sf-similar-article)
>
> [electron 在webview的iframe内加载preload - SegmentFault 思否](https://segmentfault.com/a/1190000022488251)
>
> [nodeintegrationinsubframes | Electron (electronjs.org)](https://www.electronjs.org/zh/docs/latest/api/webview-tag#nodeintegrationinsubframes) 让webview里的iframes支持node
>
> [ipcRenderer | Electron (electronjs.org)](https://www.electronjs.org/docs/latest/api/ipc-renderer)
>
> [BrowserWindow | Electron (electronjs.org)](https://www.electronjs.org/zh/docs/latest/api/browser-window)
>
> [Web Embeds | Electron (electronjs.org)](https://www.electronjs.org/zh/docs/latest/tutorial/web-embeds#webviews)
>
> [Web Embeds | Electron (electronjs.org)](https://www.electronjs.org/zh/docs/latest/tutorial/web-embeds#iframes)
>
> [webContents | Electron (electronjs.org)](https://www.electronjs.org/zh/docs/latest/api/web-contents)
>
> [webFrameMain | Electron (electronjs.org)](https://www.electronjs.org/zh/docs/latest/api/web-frame-main#methods)
>
> MessageChannel通讯[ipcRenderer | Electron (electronjs.org)](https://www.electronjs.org/zh/docs/latest/api/ipc-renderer#ipcrendererpostmessagechannel-message-transfer)

目前的解决方案待测试如下:

- webview 可以使用preload.js，可以与主进程/渲染进程进行通讯
- iframe