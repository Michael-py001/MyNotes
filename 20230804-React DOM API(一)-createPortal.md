# 20230804-React DOM API(一)-createPortal

# React DOM API

`react-dom` 包包含一些仅支持在浏览器 DOM 环境下运行的方法，不支持在 React Native 中使用。

## API 

这些 API 可以在你的组件中导入，但是很少使用：

- [`createPortal`](https://zh-hans.react.dev/reference/react-dom/createPortal) 允许你将子组件渲染到 DOM 树的不同位置。
- [`flushSync`](https://zh-hans.react.dev/reference/react-dom/flushSync) 允许你强制 React 同步刷新状态并更新 DOM。

## 入口 

`react-dom` 包提供了两个额外的入口：

- [`react-dom/client`](https://zh-hans.react.dev/reference/react-dom/client) 包含在客户端（在浏览器中）渲染 React 组件的 API。
- [`react-dom/server`](https://zh-hans.react.dev/reference/react-dom/server) 包含在服务器上渲染 React 组件的 API。

## 已弃用 API 

这些 API 将在未来的 React 主要版本中被移除。

- [`findDOMNode`](https://zh-hans.react.dev/reference/react-dom/findDOMNode) 用于查找与类式组件实例对应的最近的 DOM 节点。
- [`hydrate`](https://zh-hans.react.dev/reference/react-dom/hydrate) 可以将服务器生成的 HTML 作为浏览器 DOM 节点，并在其中渲染 React 组件。目前已被 [`hydrateRoot`](https://zh-hans.react.dev/reference/react-dom/client/hydrateRoot) 取代。
- [`render`](https://zh-hans.react.dev/reference/react-dom/render) 可以在浏览器的 DOM 元素中渲染 React 组件，目前已被 [`createRoot`](https://zh-hans.react.dev/reference/react-dom/client/createRoot) 取代。
- [`unmountComponentAtNode`](https://zh-hans.react.dev/reference/react-dom/unmountComponentAtNode) 可以从 DOM 中移除一个已挂载的 React 组件，目前已被 [`root.unmount()`](https://zh-hans.react.dev/reference/react-dom/client/createRoot#root-unmount) 取代。

# createPortal

`createPortal` 允许你将 JSX 作为 children 渲染至 DOM 的不同部分。

```jsx
<div>
  <SomeComponent />
  {createPortal(children, domNode, key?)}
</div>
```

## 参考 

### `createPortal(children, domNode, key?)` 

调用 `createPortal` 创建 portal，并传入 **JSX** 与实际渲染的**目标 DOM 节点**：

```jsx
import { createPortal } from 'react-dom';

// ...

<div>
  <p>这个子节点被放置在父节点 div 中。</p>
  {createPortal(
    <p>这个子节点被放置在 document body 中。</p>,
    document.body
  )}
</div>
```

portal 只改变 DOM 节点的所处位置。在其他方面，渲染至 portal 的 JSX 的行为表现与作为 React 组件的子节点一致。该子节点可以访问由父节点树提供的 context 对象、事件将从子节点依循 React 树冒泡到父节点。

#### 参数 

- `children`：React 可以渲染的任何内容，如 JSX 片段（`<div />` 或 `<SomeComponent />` 等等）、[Fragment](https://zh-hans.react.dev/reference/react/Fragment)（`<>...</>`）、字符串或数字，以及这些内容构成的数组。
- `domNode`：某个已经存在的 DOM 节点，例如由 `document.getElementById()` 返回的节点。在更新过程中传递不同的 DOM 节点将导致 portal 内容被重建。
- **可选参数** `key`：用作 portal [key](https://zh-hans.react.dev/learn/rendering-lists#keeping-list-items-in-order-with-key) 的独特字符串或数字。

#### 返回值 

`createPortal` 返回一个可以包含在 JSX 中或从 React 组件中返回的 React 节点。如果 React 在渲染输出中遇见它，它将把提供的 `children` 放入提供的 `domNode` 中。

#### 警告 

- portal 中的事件传播遵循 React 树而不是 DOM 树。例如点击 `<div onClick>` 内部的 portal，将触发 `onClick` 处理程序。如果这会导致意外的问题，请在 portal 内部停止事件传播，或将 portal 移动到 React 树中的上层。

## 用法 

### 渲染到 DOM 的不同部分 

*portal* 允许组件将它们的某些子元素渲染到 DOM 中的不同位置。这使得组件的一部分可以“逃脱”它所在的容器。例如组件可以在页面其余部分上方或外部显示模态对话框和提示框。

调用 `createPortal` 并传入 JSX 与  应该放置的 DOM 节点 作为参数，然后渲染返回值以创建 portal：

```jsx
import { createPortal } from 'react-dom';

function MyComponent() {
  return (
    <div style={{ border: '2px solid black' }}>
      <p>这个子节点被放置在父节点 div 中。</p>
      {createPortal(
        <p>这个子节点被放置在 document body 中。</p>,
        document.body
      )}
    </div>
  );
}
```

React 将 传递的 JSX 对应的 DOM 节点放入 提供的 DOM 节点 中。

如果没有 portal，第二个 `<p>` 将放置在父级 `<div>` 中，但 portal 会将其“传送”到 [`document.body`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/body) 中：

<iframe src="https://codesandbox.io/embed/wispy-thunder-l5cgdr?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="wispy-thunder-l5cgdr"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

请注意，第二个段落在视觉上出现在带有边框的父级 `<div>` 之外。如果你使用开发者工具检查 DOM 结构，会发现第二个 `<p>` 直接放置在 `<body>` 中：

```jsx
<body>
  <div id="root">
    ...
      <div style="border: 2px solid black">
        <p>这个子节点被放置在父节点 div 中。</p>
      </div>
    ...
  </div>
  <p>这个子节点被放置在 document body 中。</p>
</body>
```

portal 只改变 DOM 节点的所处位置。在其他方面，portal 中的 JSX 将作为实际渲染它的 React 组件的子节点。该子节点可以访问由父节点树提供的 context 对象、事件将仍然从子节点冒泡到父节点树。

### 使用 portal 渲染模态对话框 

你可以使用 portal 创建一个浮动在页面其余部分之上的模态对话框，即使呼出对话框的组件位于带有 `overflow: hidden` 或其他干扰对话框样式的容器中。

在此示例中，这两个容器具有破坏模态对话框的样式，但是渲染到 portal 中的容器不受影响，因为在 DOM 中，模态对话框不包含在父 JSX 元素内部。

<iframe src="https://codesandbox.io/embed/musing-brattain-nxpfvq?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="musing-brattain-nxpfvq"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 陷阱

> 使用 portal 时，确保应用程序的无障碍性非常重要。例如，你可能需要管理键盘焦点，以便用户可以自然进出 portal。
>
> 创建模态对话框时，请遵循 [WAI-ARIA 模态实践指南](https://www.w3.org/WAI/ARIA/apg/#dialog_modal)。如果你使用了社区包，请确保它具有无障碍性，并遵循这些指南。

### 将 React 组件渲染到非 React 服务器标记中 

如果静态或服务端渲染的网站中只有某一部分使用 React，则 portal 可能非常有用。如果你的页面使用 Rails 等服务端框架构建，则可以在静态区域（例如侧边栏）中创建交互区域。与拥有 [多个独立的 React 根](https://zh-hans.react.dev/reference/react-dom/client/createRoot#rendering-a-page-partially-built-with-react) 相比，portal 将应用程序视为一个单一的 React 树，即使它的部分在 DOM 的不同部分渲染，也可以共享状态。

<iframe src="https://codesandbox.io/embed/broken-snow-5xcr7t?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="broken-snow-5xcr7t"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 将 React 组件渲染到非 React DOM 节点 

你还可以使用 portal 来管理在 React 之外管理的 DOM 节点的内容。假设你正在集成非 React 地图小部件，并且想要在弹出窗口中渲染 React 内容，那么可以声明一个 `popupContainer` state 变量来存储要渲染到的目标 DOM 节点：

```js
const [popupContainer, setPopupContainer] = useState(null);
```

在创建第三方小部件时，存储由小部件返回的 DOM 节点，以便可以将内容渲染到其中：

```js
useEffect(() => {
  if (mapRef.current === null) {
    const map = createMapWidget(containerRef.current);
    mapRef.current = map;
    const popupDiv = addPopupToMapWidget(map);
    setPopupContainer(popupDiv);
  }
}, []);
```

这样，一旦 `popupContainer` 可用，就可以使用 `createPortal` 将 React 内容渲染到其中：

```jsx
return (
  <div style={{ width: 250, height: 250 }} ref={containerRef}>
    {popupContainer !== null && createPortal(
      <p>来自 React 的你，你好！</p>,
      popupContainer
    )}
  </div>
);
```

以下是一个完整的示例，你可以尝试一下：

<iframe src="https://codesandbox.io/embed/nervous-leaf-wlyw4j?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="nervous-leaf-wlyw4j"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>