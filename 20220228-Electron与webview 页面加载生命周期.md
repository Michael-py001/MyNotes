# Electron & webview 页面加载生命周期

## webview页面加载生命周期

1.did-start-loading    页面开始加载

2.load-commit            主页面文档加载

3.page-title-updated    title

4.dom-ready            主页面 dom 加载完成

5.load-commit            frame文档加载

6.did-frame-finish-load    frame 加载完成

7.did-frame-finish-load    最后一个是主框架frame 加载完成

8.did-finish-load            页面加载完成

9.page-favicon-updated    网页 icon

10.did-stop-loading        页面停止加载

链接：https://juejin.cn/post/6844903862806003726

## Electron的生命周期

[Electron 生命周期看这篇就够了 - 掘金 (juejin.cn)](https://juejin.cn/post/6937535140222468110)

![image-20220526134909205](https://s2.loli.net/2022/05/26/tBAGfiIPbkCxnDy.png)