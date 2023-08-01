# 20230801-ReactHooks-useDeferredValue & useEffect

## useEffect

`useEffect` 是一个 React Hook，它允许你 [将组件与外部系统同步](https://zh-hans.react.dev/learn/synchronizing-with-effects)。

```js
useEffect(setup, dependencies?)
```

### 参考 

在组件的顶层调用 `useEffect` 来声明一个 Effect：

```js
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
  // ...
}
```

### 参数 

- `setup`：处理 Effect 的函数。setup 函数选择性返回一个 **清理（cleanup）** 函数。在将组件首次添加到 DOM 之前，React 将运行 setup 函数。在每次依赖项变更重新渲染后，React 将首先使用旧值运行 cleanup 函数（如果你提供了该函数），然后使用新值运行 setup 函数。在组件从 DOM 中移除后，React 将最后一次运行 cleanup 函数。
- **可选** `dependencies`：`setup` 代码中引用的所有响应式值的列表。响应式值包括 props、state 以及所有直接在组件内部声明的变量和函数。如果你的代码检查工具 [配置了 React](https://zh-hans.react.dev/learn/editor-setup#linting)，那么它将验证是否每个响应式值都被正确地指定为一个依赖项。依赖项列表的元素数量必须是固定的，并且必须像 `[dep1, dep2, dep3]` 这样内联编写。React 将使用 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 来比较每个依赖项和它先前的值。如果省略此参数，则在每次重新渲染组件之后，将重新运行 Effect 函数。如果你想了解更多，请参见 [传递依赖数组、空数组和不传递依赖项之间的区别](https://zh-hans.react.dev/reference/react/useEffect#examples-dependencies)。

### 返回值 

`useEffect` 返回 `undefined`。

### 用法 

#### 连接到外部系统 

有些组件需要与网络、某些浏览器 API 或第三方库保持连接，当它们显示在页面上时。这些系统不受 React 控制，所以称为外部系统。

要 [将组件连接到某个外部系统](https://zh-hans.react.dev/learn/synchronizing-with-effects)，请在组件的顶层调用 `useEffect`：

```js
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
  	const connection = createConnection(serverUrl, roomId);
    connection.connect();
  	return () => {
      connection.disconnect();
  	};
  }, [serverUrl, roomId]);
  // ...
}
```

你需要向 `useEffect` 传递两个参数：

1. 一个 **setup 函数** ，其 **setup 代码** 用来连接到该系统。
   - 它应该返回一个 **清理函数**（cleanup），其 cleanup 代码 用来与该系统断开连接。
2. 一个 **依赖项列表**，包括这些函数使用的每个组件内的值。

**React 在必要时会调用 setup 和 cleanup，这可能会发生多次**：

1. 将组件挂载到页面时，将运行 **setup 代码**。
2. 重新渲染 **依赖项** 变更的组件后：
   - 首先，使用旧的 props 和 state 运行 **cleanup 代码**
   - 然后，使用新的 props 和 state 运行 **setup 代码**。
3. 当组件从页面卸载后，**cleanup 代码** 将运行最后一次

**用上面的代码作为例子来解释这个顺序**。

当 `ChatRoom` 组件添加到页面中时，它将使用 `serverUrl` 和 `roomId` 初始值连接到聊天室。如果 `serverUrl` 或者 `roomId` 发生改变并导致重新渲染（比如用户在下拉列表中选择了一个不同的聊天室），那么 Effect 就会 **断开与前一个聊天室的连接，并连接到下一个聊天室**。当 `ChatRoom` 组件从页面中卸载时，你的 Effect 将最后一次断开连接。

**为了 [帮助你发现 bug](https://zh-hans.react.dev/learn/synchronizing-with-effects#step-3-add-cleanup-if-needed)，在开发环境下，React 在运行 setup 之前会额外运行一次setup 和 cleanup**。这是一个压力测试，用于验证 Effect 逻辑是否正确实现。如果这会导致可见的问题，那么你的 cleanup 函数就缺少一些逻辑。cleanup 函数应该停止或撤消 setup 函数正在执行的任何操作。一般来说，用户不应该能够区分只调用一次 setup（在生产环境中）与调用 *setup* → *cleanup* → *setup* 序列（在开发环境中）。[查看常见解决方案](https://zh-hans.react.dev/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)。

**尽量 [将每个 Effect 作为一个独立的过程编写](https://zh-hans.react.dev/learn/lifecycle-of-reactive-effects#each-effect-represents-a-separate-synchronization-process)，并且 [每次只考虑一个单独的 setup/cleanup 周期](https://zh-hans.react.dev/learn/lifecycle-of-reactive-effects#thinking-from-the-effects-perspective)**。组件是否正在挂载、更新或卸载并不重要。当你的 cleanup 逻辑正确地“映射”到 setup 逻辑时，你的 Effect 是可复原的，因此可以根据需要多次运行 setup 和 cleanup 函数。

> ### 注意
>
> Effect 可以让你的组件与某些外部系统（比如聊天服务）[保持同步](https://zh-hans.react.dev/learn/synchronizing-with-effects)。在这里，外部系统是指任何不受 React 控制的代码，例如：
>
> - 由 [`setInterval()`](https://developer.mozilla.org/zh-CN/docs/Web/API/setInterval) 和 [`clearInterval()`](https://developer.mozilla.org/zh-CN/docs/Web/API/clearInterval) 管理的定时器。
> - 使用 [`window.addEventListener()`](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/addEventListener) 和 [`window.removeEventListener()`](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/removeEventListener) 的事件订阅。
> - 一个第三方的动画库，它有一个类似 `animation.start()` 和 `animation.reset()` 的 API。
>
> **如果你没有连接到任何外部系统，[你或许不需要 Effect](https://zh-hans.react.dev/learn/you-might-not-need-an-effect)**。

## useDeferredValue

`useDeferredValue` 是一个 React Hook，可以让你延迟更新 UI 的某些部分。

```js
const deferredValue = useDeferredValue(value)
```

### 参数 

- `value`：你想延迟的值，可以是任何类型。

### 返回值 

在组件的初始渲染期间，返回的延迟值将与你提供的值相同。但是在组件更新时，React 将会先尝试使用旧值进行重新渲染（因此它将返回旧值），然后再在后台使用新值进行另一个重新渲染（这时它将返回更新后的值）。

### 注意事项 

- 你应该向 `useDeferredValue` 传递原始值（如字符串和数字）或在渲染之外创建的对象。如果你在渲染期间创建了一个新对象，并立即将其传递给 `useDeferredValue`，那么每次渲染时这个对象都会不同，这将导致后台不必要的重新渲染。
- 当 `useDeferredValue` 接收到与之前不同的值（使用 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 进行比较）时，除了当前渲染（此时它仍然使用旧值），它还会安排一个后台重新渲染。这个后台重新渲染是可以被中断的，如果 `value` 有新的更新，React 会从头开始重新启动后台渲染。举个例子，如果用户在输入框中的输入速度比接收延迟值的图表重新渲染的速度快，那么图表只会在用户停止输入后重新渲染。
- `useDeferredValue` 与 [``](https://zh-hans.react.dev/reference/react/Suspense) 集成。如果由于新值引起的后台更新导致 UI 暂停，用户将不会看到 fallback 效果。他们将看到旧的延迟值，直到数据加载完成。
- `useDeferredValue` 本身并不能阻止额外的网络请求。
- `useDeferredValue` 本身不会引起任何固定的延迟。一旦 React 完成原始的重新渲染，它会立即开始使用新的延迟值处理后台重新渲染。由事件（例如输入）引起的任何更新都会中断后台重新渲染，并被优先处理。
- 由 `useDeferredValue` 引起的后台重新渲染在提交到屏幕之前不会触发 Effect。如果后台重新渲染被暂停，Effect 将在数据加载后和 UI 更新后运行。

### 用法 

#### 在新内容加载期间显示旧内容。 

在组件的顶层调用 `useDeferredValue` 来延迟更新 UI 的某些部分。

```js
import { useState, useDeferredValue } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  // ...
}
```

在初始渲染期间，返回的 延迟值 与你提供的 值 相同。

在更新期间，延迟值 会“滞后于”最新的 值。具体地说，React 首先会在不更新延迟值的情况下进行重新渲染，然后在后台尝试使用新接收到的值进行重新渲染。

**让我们通过一个例子来看看什么时候该使用它**。

> ### 注意
>
> 这个例子假设你使用了其中一个支持 `Suspense` 的数据源：
>
> - 使用支持 suspense 的框架进行数据获取，例如 [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) 和 [Next.js](https://nextjs.org/docs/getting-started/react-essentials)
> - 使用 [`lazy`](https://zh-hans.react.dev/reference/react/lazy) 懒加载组件代码
>
> [了解更多有关 suspense 及其限制的信息](https://zh-hans.react.dev/reference/react/Suspense)。

这个例子中，在获取搜索结果时，`SearchResults` 组件会 [suspend](https://zh-hans.react.dev/reference/react/Suspense#displaying-a-fallback-while-content-is-loading)。尝试输入 `"a"`，等待结果出现后，将其编辑为 `"ab"`。此时 `"a"` 的结果会被加载中的 fallback 替代。

<iframe src="https://codesandbox.io/embed/peaceful-pine-mjgp6r?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="peaceful-pine-mjgp6r"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

一个常见的备选 UI 模式是 **延迟** 更新结果列表，并继续显示之前的结果，直到新的结果准备好。调用 `useDeferredValue` 并将延迟版本的查询参数向下传递：

```jsx
export default function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  return (
    <>
      <label>
        Search albums:
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <SearchResults query={deferredQuery} />
      </Suspense>
    </>
  );
}
```

`query` 会立即更新，所以输入框将显示新值。然而，`deferredQuery` 在数据加载完成前会保留以前的值，因此 `SearchResults` 将暂时显示旧的结果。

在下面的示例中，输入 `"a"`，等待结果加载完成，然后将输入框编辑为 `"ab"`。注意，现在你看到的不是 suspense fallback，而是旧的结果列表，直到新的结果加载完成：

<iframe src="https://codesandbox.io/embed/crazy-meitner-rjz9nm?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="crazy-meitner-rjz9nm"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

##### 深入探讨：如何在内部实现延迟值？ 

你可以将其看成两个步骤：

1. **首先，React 会使用新的 `query` 值（`"ab"`）和旧的 `deferredQuery` 值（仍为 `"a"`）重新渲染。** 传递给结果列表的 `deferredQuery` 值是**延迟**的，它“滞后于” `query` 值。
2. **在后台，React 尝试重新渲染，并将 `query` 和 `deferredQuery` 两个值都更新为 `"ab"`。** 如果此次重新渲染完成，React 将在屏幕上显示它。但是，如果它 suspense（即 `"ab"` 的结果尚未加载），React 将放弃这次渲染，并在数据加载后再次尝试重新渲染。用户将一直看到旧的延迟值，直到数据准备就绪。

被推迟的“后台”渲染是可中断的。例如，如果你再次在输入框中输入，React 将会中断渲染，并从新值开始重新渲染。React 总是使用最新提供的值。

注意，每次按键仍会发起一个网络请求。这里延迟的是显示结果（直到它们准备就绪），而不是网络请求本身。即使用户继续输入，每个按键的响应都会被缓存，所以按下 Backspace 键是瞬时的，不会再次获取数据。

#### 表明内容已过时

在上面的示例中，当最新的查询结果仍在加载时，没有任何提示。如果新的结果需要一段时间才能加载完成，这可能会让用户感到困惑。为了更明显地告知用户结果列表与最新查询不匹配，你可以在显示旧的查询结果时添加一个视觉提示：

```jsx
<div style={{
  opacity: query !== deferredQuery ? 0.5 : 1,
}}>
  <SearchResults query={deferredQuery} />
</div>
```

有了上面这段代码，当你开始输入时，旧的结果列表会略微变暗，直到新的结果列表加载完毕。你也可以添加 CSS 过渡来延迟变暗的过程，让用户感受到一种渐进式的过渡，就像下面的例子一样：

<iframe src="https://codesandbox.io/embed/trusting-sanderson-g2zdfs?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="trusting-sanderson-g2zdfs"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

#### 延迟渲染 UI 的某些部分 

你还可以将 `useDeferredValue` 作为性能优化的手段。当你的 UI 某个部分重新渲染很慢、没有简单的优化方法，同时你又希望避免它阻塞其他 UI 的渲染时，使用 `useDeferredValue` 很有帮助。

想象一下，你有一个文本框和一个组件（例如图表或长列表），在每次按键时都会重新渲染：

```jsx
function App() {
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <SlowList text={text} />
    </>
  );
}
```

首先，我们可以优化 `SlowList`，使其在 props 不变的情况下跳过重新渲染。只需将其 [用 `memo` 包裹](https://zh-hans.react.dev/reference/react/memo#skipping-re-rendering-when-props-are-unchanged) 即可：

```js
const SlowList = memo(function SlowList({ text }) {
  // ...
});
```

然而，这仅在 `SlowList` 的 props 与上一次的渲染时相同才有用。你现在遇到的问题是，当这些 props **不同** 时，并且实际上需要展示不同的视觉输出时，页面会变得很慢。

具体而言，主要的性能问题在于，每次你输入内容时，`SlowList` 都会接收新的 props，并重新渲染整个树结构，这会让输入感觉很卡顿。使用 `useDeferredValue` 能够优先更新输入框（必须快速更新），而不是更新结果列表（可以更新慢一些），从而缓解这个问题：

```jsx
function App() {
  const [text, setText] = useState('');
  const deferredText = useDeferredValue(text);
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <SlowList text={deferredText} />
    </>
  );
}
```

这并没有让 `SlowList` 的重新渲染变快。然而，它告诉 React 可以将列表的重新渲染优先级降低，这样就不会阻塞按键输入。列表的更新会“滞后”于输入，然后“追赶”上来。与之前一样，React 会尽快更新列表，但不会阻塞用户输入。

优化后：

> 在这个例子中，`SlowList` 组件中的每个 item 都被 **故意减缓了渲染速度**，这样你就可以看到 `useDeferredValue` 是如何让输入保持响应的。当你在输入框中输入时，你会发现输入很灵敏，而列表的更新会稍有延迟。

<iframe src="https://codesandbox.io/embed/competent-architecture-ckv4sc?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="competent-architecture-ckv4sc"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

未优化：

> 在这个例子中，`SlowList` 组件中的每个 item 都被 **故意减缓了渲染速度**，但这次没有使用 `useDeferredValue`。
>
> 注意，输入框的输入感觉非常卡顿。这是因为没有使用 `useDeferredValue`，每次按键都会立即强制整个列表以不可中断的方式进行重新渲染。

<iframe src="https://codesandbox.io/embed/angry-fast-tzyqs4?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="angry-fast-tzyqs4"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

##### 陷阱

这个优化需要将 `SlowList` 包裹在 [`memo`](https://zh-hans.react.dev/reference/react/memo) 中。这是因为每当 `text` 改变时，React 需要能够快速重新渲染父组件。在重新渲染期间，`deferredText` 仍然保持着之前的值，因此 `SlowList` 可以跳过重新渲染（它的 props 没有改变）。如果没有 [`memo`](https://zh-hans.react.dev/reference/react/memo)，`SlowList` 仍会重新渲染，这将使优化失去意义。

##### 深入探讨： 延迟一个值与防抖和节流之间有什么不同？ 

在上述的情景中，你可能会使用这两种常见的优化技术：

- **防抖** 是指在用户停止输入一段时间（例如一秒钟）之后再更新列表。
- **节流** 是指每隔一段时间（例如最多每秒一次）更新列表。

虽然这些技术在某些情况下是有用的，但 `useDeferredValue` 更适合优化渲染，因为它与 React 自身深度集成，并且能够适应用户的设备。

与防抖或节流不同，`useDeferredValue` 不需要选择任何固定延迟时间。如果用户的设备很快（比如性能强劲的笔记本电脑），延迟的重渲染几乎会立即发生并且不会被察觉。如果用户的设备较慢，那么列表会相应地“滞后”于输入，滞后的程度与设备的速度有关。

此外，与防抖或节流不同，`useDeferredValue` 执行的延迟重新渲染默认是可中断的。这意味着，如果 React 正在重新渲染一个大型列表，但用户进行了另一次键盘输入，React 会放弃该重新渲染，先处理键盘输入，然后再次开始在后台渲染。相比之下，防抖和节流仍会产生不顺畅的体验，因为它们是阻塞的：它们仅仅是将渲染阻塞键盘输入的时刻推迟了。

如果你要优化的工作不是在渲染期间发生的，那么防抖和节流仍然非常有用。例如，它们可以让你减少网络请求的次数。你也可以同时使用这些技术。