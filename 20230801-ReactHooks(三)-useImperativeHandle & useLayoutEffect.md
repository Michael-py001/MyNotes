# 20230801-ReactHooks-useImperativeHandle & useLayoutEffect

## useImperativeHandle 

`useImperativeHandle` 是 React 中的一个 Hook，它能让你自定义由 [ref](https://zh-hans.react.dev/learn/manipulating-the-dom-with-refs) 暴露出来的句柄。

```js
useImperativeHandle(ref, createHandle, dependencies?)
```

### 参考 

`useImperativeHandle(ref, createHandle, dependencies?)` 

在组件顶层通过调用 `useImperativeHandle` 来自定义 ref 暴露出来的句柄，通常搭配`forwardRef`使用。

```js
import { forwardRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  useImperativeHandle(ref, () => {
    return {
      // ... 你的方法 ...
    };
  }, []);
  // ...
```

### 参数 

- `ref`：该 `ref` 是你从 [`forwardRef` 渲染函数](https://zh-hans.react.dev/reference/react/forwardRef#render-function) 中获得的第二个参数。
- `createHandle`：该函数无需参数，它返回你想要暴露的 ref 的句柄。该句柄可以包含任何类型。通常，你会返回一个包含你想暴露的方法的对象。
- **可选的** `dependencies`：函数 `createHandle` 代码中所用到的所有反应式的值的列表。反应式的值包含 props、状态和其他所有直接在你组件体内声明的变量和函数。倘若你的代码检查器已 [为 React 配置好](https://zh-hans.react.dev/learn/editor-setup#linting)，它会验证每一个反应式的值是否被正确指定为依赖项。该列表的长度必须是一个常数项，并且必须按照 `[dep1, dep2, dep3]` 的形式罗列各依赖项。React 会使用 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 来比较每一个依赖项与其对应的之前值。如果一次重新渲染导致某些依赖项发生了改变，或你没有提供这个参数列表，你的函数 `createHandle` 将会被重新执行，而新生成的句柄则会被分配给 ref。

### 返回值 

`useImperativeHandle` 返回 `undefined`。

### 使用方法 

#### 向父组件暴露一个自定义的 ref 句柄 

默认情况下，组件不会将它们的 DOM 节点暴露给父组件。举例来说，如果你想要 `MyInput` 的父组件 [能访问到](https://zh-hans.react.dev/learn/manipulating-the-dom-with-refs) `<input>` DOM 节点，你必须选择使用 [`forwardRef`:](https://zh-hans.react.dev/reference/react/forwardRef)。

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  return <input {...props} ref={ref} />;
});
```

在上方的代码中，[`MyInput` 的 ref 会接收到 ` DOM 节点`](https://zh-hans.react.dev/reference/react/forwardRef#exposing-a-dom-node-to-the-parent-component)。然而，你可以选择暴露一个自定义的值。为了修改被暴露的句柄，在你的顶层组件调用 `useImperativeHandle`：

```jsx
import { forwardRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  useImperativeHandle(ref, () => {
    return {
      // ... 你的方法 ...
    };
  }, []);

  return <input {...props} />;
});
```

举例来说，假设你不想暴露出整个 `<input>` DOM 节点，但你想要它其中两个方法：`focus` 和 `scrollIntoView`。为此，用单独额外的 ref 来指向真实的浏览器 DOM。然后使用 `useImperativeHandle` 来暴露一个句柄，它只返回你想要父组件去调用的方法：

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => {
    return {
      focus() {
        inputRef.current.focus();
      },
      scrollIntoView() {
        inputRef.current.scrollIntoView();
      },
    };
  }, []);

  return <input {...props} ref={inputRef} />;
});
```

现在，如果你的父组件获得了 `MyInput` 的 ref，就能通过该 ref 来调用 `focus` 和 `scrollIntoView` 方法。然而，它的访问是受限的，无法读取或调用下方 `<input>` DOM 节点的其他所有属性和方法。

<iframe src="https://codesandbox.io/embed/wandering-sound-zlxuek?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="wandering-sound-zlxuek"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

#### 暴露你自己的命令式方法

你通过命令式句柄暴露出来的方法不一定需要完全匹配 DOM 节点的方法。例如，这个 `Post` 组件暴露了一个 `scrollAndFocusAddComment` 方法。它可以让你在点击按钮后，使父组件 `Page` 滚动到评论列表的底部 *并* 聚焦到输入框：

<iframe src="https://codesandbox.io/embed/inspiring-lake-50qjw4?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="inspiring-lake-50qjw4"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 陷阱

**不要滥用 ref。** 你应当仅在你没法通过 prop 来表达 *命令式* 行为的时候才使用 ref：例如，滚动到指定节点、聚焦某个节点、触发一次动画，以及选择文本等等。

**如果可以通过 prop 实现，那就不应该使用 ref**。例如，你不应该从一个 `Model` 组件暴露出 `{open, close}` 这样的命令式句柄，最好是像 `<Modal isOpen={isOpen} />` 这样，将 `isOpen` 作为一个 prop。[副作用](https://zh-hans.react.dev/learn/synchronizing-with-effects) 可以帮你通过 prop 来暴露一些命令式的行为。

> 将 `isOpen` 作为 `Modal` 组件的 `prop`，而不是暴露出命令式句柄，有以下好处：
>
> 1. 提高组件的可复用性：将 `isOpen` 作为 `prop`，可以使 `Modal` 组件更加通用和可复用。因为 `isOpen` 是一个普通的 `prop`，可以通过父组件来控制 `Modal` 组件的显示和隐藏，而不需要暴露出命令式句柄。这样，`Modal` 组件可以在不同的场景中被复用，而不需要修改其实现细节。
> 2. 提高组件的可测试性：将 `isOpen` 作为 `prop`，可以使 `Modal` 组件更容易进行单元测试。因为 `isOpen` 是一个普通的 `prop`，可以通过测试代码来控制 `Modal` 组件的显示和隐藏，而不需要依赖于命令式句柄。这样，可以更容易地编写测试用例，测试 `Modal` 组件的各种状态和行为。
> 3. 提高代码的可读性：将 `isOpen` 作为 `prop`，可以使代码更加易于理解和维护。因为 `isOpen` 是一个普通的 `prop`，可以通过查看组件的 `prop` 定义，了解组件的行为和状态。而如果使用命令式句柄，代码的含义就不够明确，需要查看句柄的实现细节才能理解组件的行为和状态。
>
> 综上所述，将 `isOpen` 作为 `Modal` 组件的 `prop`，可以提高组件的可复用性、可测试性和代码的可读性，是一种更好的实现方式。



# useLayoutEffect

> ### 陷阱
>
> `useLayoutEffect` 可能会影响性能。尽可能使用 [`useEffect`](https://zh-hans.react.dev/reference/react/useEffect)。

`useLayoutEffect` 是 [`useEffect`](https://zh-hans.react.dev/reference/react/useEffect) 的一个版本，在浏览器重新绘制屏幕之前触发。

```js
useLayoutEffect(setup, dependencies?)
```

### 参考 

`useLayoutEffect(setup, dependencies?)` 

调用 `useLayoutEffect` 在浏览器重新绘制屏幕之前进行布局测量：

```js
import { useState, useRef, useLayoutEffect } from 'react';

function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height);
  }, []);
  // ...
```

### 参数 

- `setup`：处理副作用的函数。setup 函数选择性返回一个*清理*（cleanup）函数。在将组件首次添加到 DOM 之前，React 将运行 setup 函数。在每次因为依赖项变更而重新渲染后，React 将首先使用旧值运行 cleanup 函数（如果你提供了该函数），然后使用新值运行 setup 函数。在组件从 DOM 中移除之前，React 将最后一次运行 cleanup 函数。
- **可选** `dependencies`：`setup` 代码中引用的所有响应式值的列表。响应式值包括 props、state 以及所有直接在组件内部声明的变量和函数。如果你的代码检查工具 [配置了 React](https://zh-hans.react.dev/learn/editor-setup#linting)，那么它将验证每个响应式值都被正确地指定为一个依赖项。依赖项列表必须具有固定数量的项，并且必须像 `[dep1, dep2, dep3]` 这样内联编写。React 将使用 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 来比较每个依赖项和它先前的值。如果省略此参数，则在每次重新渲染组件之后，将重新运行副作用函数。

### 返回值 

`useLayoutEffect` 返回 `undefined`。

### 注意事项 

- `useLayoutEffect` 是一个 Hook，因此只能在 **组件的顶层** 或自己的 Hook 中调用它。不能在循环或者条件内部调用它。如果你需要的话，抽离出一个组件并将副作用处理移动到那里。
- 当 StrictMode 启用时，React 将在真正的 setup 函数首次运行前，**运行一个额外的开发专有的 setup + cleanup 周期**。这是一个压力测试，确保 cleanup 逻辑“映照”到 setup 逻辑，并停止或撤消 setup 函数正在做的任何事情。如果这导致一个问题，[请实现清理函数](https://zh-hans.react.dev/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)。
- 如果你的一些依赖项是组件内部定义的对象或函数，则存在这样的风险，即它们将 **导致 Effect 重新运行的次数多于所需的次数**。要解决这个问题，请删除不必要的 [对象](https://zh-hans.react.dev/reference/react/useEffect#removing-unnecessary-object-dependencies) 和 [函数](https://zh-hans.react.dev/reference/react/useEffect#removing-unnecessary-function-dependencies) 依赖项。你还可以 [抽离状态更新](https://zh-hans.react.dev/reference/react/useEffect#updating-state-based-on-previous-state-from-an-effect) 和 [非响应式逻辑](https://zh-hans.react.dev/reference/react/useEffect#reading-the-latest-props-and-state-from-an-effect) 到 Effect 之外。
- Effect **只在客户端上运行**，在服务端渲染中不会运行。
- `useLayoutEffect` 内部的代码和所有计划的状态更新阻塞了浏览器重新绘制屏幕。如果过度使用，这会使你的应用程序变慢。如果可能的话，尽量选择 [`useEffect`](https://zh-hans.react.dev/reference/react/useEffect)。

### 用法 

#### 在浏览器重新绘制屏幕前计算布局 

大多数组件不需要依靠它们在屏幕上的位置和大小来决定渲染什么。他们只返回一些 JSX，然后浏览器计算他们的 **布局**（位置和大小）并重新绘制屏幕。

有时候，这还不够。想象一下悬停时出现在某个元素旁边的 tooltip。如果有足够的空间，tooltip 应该出现在元素的上方，但是如果不合适，它应该出现在下面。为了让 tooltip 渲染在最终正确的位置，你需要知道它的高度（即它是否适合放在顶部）。

要做到这一点，你需要分两步渲染：

1. 将 tooltip 渲染到任何地方（即使位置不对）。
2. 测量它的高度并决定放置 tooltip 的位置。
3. 把 tooltip 渲染放在正确的位置。

**所有这些都需要在浏览器重新绘制屏幕之前完成**。你不希望用户看到 tooltip 在移动。调用 `useLayoutEffect` 在浏览器重新绘制屏幕之前执行布局测量：

```js
function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0); // 你还不知道真正的高度

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height); // 现在重新渲染，你知道了真实的高度
  }, []);

  // ... 在下方的渲染逻辑中使用 tooltipHeight ...
}
```

下面是这如何一步步工作的：

1. `Tooltip` 使用初始值 `tooltipHeight = 0` 进行渲染（因此 tooltip 可能被错误地放置）。
2. React 将它放在 DOM 中，然后运行 `useLayoutEffect` 中的代码。
3. `useLayoutEffect` [测量](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect) 了 tooltip 内容的高度，并立即触发重新渲染。
4. 使用实际的 `tooltipHeight` 再次渲染 `Tooltip`（这样 tooltip 的位置就正确了）。
5. React 在 DOM 中对它进行更新，浏览器最终显示出 tooltip。

将鼠标悬停在下面的按钮上，看看 tooltip 是如何根据它是否合适来调整它的位置：

<iframe src="https://codesandbox.io/embed/admiring-volhard-v5c33z?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="admiring-volhard-v5c33z"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

注意，即使 `Tooltip` 组件需要两次渲染（首先，使用初始值为 0 的 `tooltipHeight` 渲染，然后使用实际测量的高度渲染），你也只能看到最终结果。这就是为什么在这个例子中需要 `useLayoutEffect` 而不是 [`useEffect`](https://zh-hans.react.dev/reference/react/useEffect) 的原因。让我们来看看下面的细节差别。

为了使 bug 更容易重现，此版本在渲染期间人为地添加了延迟。React 将在处理 `useEffect` 内部的状态更新之前让浏览器绘制屏幕。结果，tooltip 会闪烁：

<iframe src="https://codesandbox.io/embed/kind-dubinsky-t673cw?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="kind-dubinsky-t673cw"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

使用 `useLayoutEffect` 编辑这个例子，可以观察到即使渲染速度减慢，它也会阻塞绘制。

> ### 注意
>
> 两次渲染并阻塞浏览器绘制会影响性能。尽量避免这种情况。