# 20240902-iframe的onload事件不会触发的问题解决

## 问题&解决

最近的老项目发现了一个打印问题，用户的浏览器点击打印按钮触发不了浏览器的打印。经过一番排查后，发现是iframe没有触发onload事件，但是在我电脑的Edge浏览器就能触发，这就很诡异。

代码如下：

```js
const ID = 'print-iframe'
const $iframeEl = document.querySelector('#' + ID)
if ($iframeEl) {
  document.body.removeChild($iframeEl)
}
const iframe = document.createElement('iframe')
document.body.appendChild(iframe)
iframe.style.display = 'none'
iframe.onload = function () {
  setTimeout(() => {
    iframe.contentWindow.print()
    iframe.contentWindow.onafterprint = function () {
      document.body.removeChild(iframe)
    }
  }, 1000)
}
```

这段`iframe.onload`没有触发，最终问了AI才知道要设置`iframe.srcdoc`才能触发onload，但是我的电脑浏览器不加也能触发导致一开始没发现这个问题。

刚开始还加了以下代码:

```js
if (iframe.contentDocument) {
  iframe.contentDocument.body.innerHTML = html
  iframe.contentWindow.document.title = name
}
```

但是可以直接使用srcdoc属性设置，这段代码就不需要了。

到此，问题解决。

## srcdoc属性

iframe.srcdoc 是一个 HTML 属性，用于直接设置嵌入在 <iframe> 元素中的 HTML 内容。与 src 属性不同，srcdoc 属性允许你直接在 iframe 内嵌入 HTML，而不需要通过 URL 加载外部资源。

### 示例

#### 使用 srcdoc

```html
<iframe srcdoc="<p>这是嵌入的 HTML 内容</p>"></iframe>
```

在这个示例中，iframe 将直接显示 <p>这是嵌入的 HTML 内容</p>。

#### 使用 src

```html
<iframe src="https://example.com"></iframe>
```

在这个示例中，[`iframe`](vscode-file://vscode-app/c:/Users/WuShiLi/AppData/Local/Programs/Microsoft VS Code Insiders/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html) 将加载并显示 `https://example.com` 页面。

### 主要区别

1. **内容来源**：
   - [`srcdoc`](vscode-file://vscode-app/c:/Users/WuShiLi/AppData/Local/Programs/Microsoft VS Code Insiders/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html)：直接嵌入 HTML 内容。
   - [`src`](vscode-file://vscode-app/c:/Users/WuShiLi/AppData/Local/Programs/Microsoft VS Code Insiders/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html)：通过 URL 加载外部资源。
2. **加载方式**：
   - [`srcdoc`](vscode-file://vscode-app/c:/Users/WuShiLi/AppData/Local/Programs/Microsoft VS Code Insiders/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html)：不需要网络请求，直接使用嵌入的 HTML 字符串。
   - [`src`](vscode-file://vscode-app/c:/Users/WuShiLi/AppData/Local/Programs/Microsoft VS Code Insiders/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html)：需要网络请求来加载指定的 URL。
3. **使用场景**：
   - [`srcdoc`](vscode-file://vscode-app/c:/Users/WuShiLi/AppData/Local/Programs/Microsoft VS Code Insiders/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html)：适用于需要动态生成或嵌入 HTML 内容的场景。
   - [`src`](vscode-file://vscode-app/c:/Users/WuShiLi/AppData/Local/Programs/Microsoft VS Code Insiders/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html)：适用于加载外部网页或资源的场景。

