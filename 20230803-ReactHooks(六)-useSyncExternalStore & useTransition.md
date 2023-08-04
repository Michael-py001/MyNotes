# 20230803-ReactHooks-useSyncExternalStore & useTransition

# useSyncExternalStore

`useSyncExternalStore` 是一个让你订阅外部 store 的 React Hook。

```js
const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)
```

## 参考 

### `useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)` 

在组件顶层调用 `useSyncExternalStore` 以从外部 store 读取值。

```js
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  // ...
}
```

它返回 store 中数据的快照。你需要传两个函数作为参数：

1. `subscribe` 函数应当订阅该 store 并返回一个取消订阅的函数。
2. `getSnapshot` 函数应当从该 store 读取数据的快照。

#### 参数 

- `subscribe`：一个函数，接收一个单独的 `callback` 参数并把它订阅到 store 上。当 store 发生改变，它应当调用被提供的 `callback`。这会导致组件重新渲染。`subscribe` 函数会返回清除订阅的函数。
- `getSnapshot`：一个函数，返回组件需要的 store 中的数据快照。在 store 不变的情况下，重复调用 `getSnapshot` 必须返回同一个值。如果 store 改变，并且返回值也不同了（用 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较），React 就会重新渲染组件。
- **可选** `getServerSnapshot`：一个函数，返回 store 中数据的初始快照。它只会在服务端渲染时，以及在客户端进行服务端渲染内容的 hydration 时被用到。快照在服务端与客户端之间必须相同，它通常是从服务端序列化并传到客户端的。如果你忽略此参数，在服务端渲染这个组件会抛出一个错误。

#### 返回值 

该 store 的当前快照，可以在你的渲染逻辑中使用。

#### 警告 

- `getSnapshot` 返回的 store 快照必须是不可变的。如果底层 store 有可变数据，要在数据改变时返回一个新的不可变快照。否则，返回上次缓存的快照。
- 如果在重新渲染时传入一个不同的 `subscribe` 函数，React 会用新传入的 `subscribe` 函数重新订阅该 store。你可以通过在组件外声明 `subscribe` 来避免。

## 使用 

### 订阅外部 store 

你的多数组件只会从它们的 [props](https://zh-hans.react.dev/learn/passing-props-to-a-component)，[state](https://zh-hans.react.dev/reference/react/useState)，以及 [context](https://zh-hans.react.dev/reference/react/useContext) 读取数据。然而，有时一个组件需要从一些 React 之外的 store 读取一些随时间变化的数据。这包括：

- 在 React 之外持有状态的第三方状态管理库
- 暴露出一个可变值及订阅其改变事件的浏览器 API

在组件顶层调用 `useSyncExternalStore` 以从外部 store 读取值。

```js
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  // ...
}
```

它返回 store 中数据的 **快照**。你需要传两个函数作为参数：

1. `subscribe` 函数应当订阅 store 并返回一个取消订阅函数。
2. `getSnapshot` 函数应当从 store 中读取数据的快照。

React 会用这些函数来保持你的组件订阅到 store 并在它改变时**重新渲染**。

例如，在下面的沙盒中，`todosStore` 被实现为一个保存 React 之外数据的外部 store。`TodosApp` 组件通过 `useSyncExternalStore` Hook 连接到**外部 store**。

<iframe src="https://codesandbox.io/embed/amazing-ben-m3k8zq?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="amazing-ben-m3k8zq"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

> ### 注意
>
> 当可能的时候，我们推荐通过 [`useState`](https://zh-hans.react.dev/reference/react/useState) 和 [`useReducer`](https://zh-hans.react.dev/reference/react/useReducer) 使用内建的 React state 代替。如果你需要去集成已有的非 React 代码，`useSyncExternalStore` API 是很有用的。

### 订阅浏览器 API 

添加 `useSyncExternalStore` 的另一个场景是当你想订阅一些由浏览器暴露的并随时间变化的值时。例如，假设你想要组件展示网络连接是否正常。浏览器通过一个叫做 [`navigator.onLine`](https://developer.mozilla.org/zh-CN/docs/Web/API/Navigator/onLine) 的属性暴露出这一信息。

这个值可能在 React 不知道的情况下改变，所以你应当通过 `useSyncExternalStore` 来读取它。

```js
import { useSyncExternalStore } from 'react';

function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  // ...
}
```

从浏览器 API 读取当前值，来实现 `getSnapshot` 函数：

```js
function getSnapshot() {
  return navigator.onLine;
}
```

接下来，你需要实现 `subscribe` 函数。例如，当 `navigator.onLine` 改变，浏览器触发 `window` 对象的 [`online`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/online_event) 和 [`offline`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/offline_event) 事件，然后返回一个清除订阅的函数：

```jsx
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```

现在 React 知道如何从外部的 `navigator.onLine` API 读到这个值，以及如何订阅其改变。断开你的设备的网络，就可以观察到组件重新渲染了：

<iframe src="https://codesandbox.io/embed/loving-yalow-e010i8?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="loving-yalow-e010i8"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 把逻辑抽取到自定义 Hook 

通常不会在组件里直接用 `useSyncExternalStore`，而是在自定义 Hook 里调用它。这使得你可以在不同组件里使用相同的外部 store。

例如：这里自定义的 `useOnlineStatus` Hook 追踪网络是否在线：

```js
import { useSyncExternalStore } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return isOnline;
}

function getSnapshot() {
  // ...
}

function subscribe(callback) {
  // ...
}
```

现在不同的组件都可以调用 `useOnlineStatus`，而不必重复底层实现：

<iframe src="https://codesandbox.io/embed/objective-water-m59nhx?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="objective-water-m59nhx"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 添加服务端渲染支持 

如果你的 React 应用使用 [服务端渲染](https://zh-hans.react.dev/reference/react-dom/server)，你的 React 组件也会运行在浏览器环境之外来生成初始 HTML。这给连接到外部 store 造成了一些挑战：

- 如果你连接到一个浏览器特有的 API，因为它在服务端不存在，所以是不可行的。
- 如果你连接到一个第三方 store，数据要在服务端和客户端之间相匹配。

为了解决这些问题，要传一个 `getServerSnapshot` 函数作为第三个参数给 `useSyncExternalStore`：

```js
import { useSyncExternalStore } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
  return isOnline;
}

function getSnapshot() {
  return navigator.onLine;
}

function getServerSnapshot() {
  return true; // 服务端生成的 HTML 总是显示“在线”
}

function subscribe(callback) {
  // ...
}
```

`getServerSnapshot` 函数与 `getSnapshot` 相似，但它只在两种情况下才运行：

- 在服务端生成 HTML 时。
- 在客户端 [hydration](https://zh-hans.react.dev/reference/react-dom/client/hydrateRoot) 时，即：当 React 拿到服务端的 HTML 并使其可交互。

这使得你能提供在应用可交互前可用的初始快照值。如果没有对服务器端渲染来说有意义的初始值，就省略这个参数来 [强制客户端渲染](https://zh-hans.react.dev/reference/react/Suspense#providing-a-fallback-for-server-errors-and-server-only-content)。

### 注意

> 确保客户端初始渲染与服务端渲染时 `getServerSnapshot` 返回完全相同的数据。例如，如果在服务端 `getServerSnapshot` 返回一些预先载入的 store 内容，你就需要把这些内容也传给客户端。一种方法是在服务端渲染时，生成 `<script>` 标签来设置像 `window.MY_STORE_DATA` 这样的全局变量，并在客户端 `getServerSnapshot` 内读取此全局变量。你的外部 store 应当提供如何这样做的说明。

## 疑难解答 

### 遇到一个错误：“The result of `getSnapshot` should be cached” 

这个错误意味着你的 `getSnapshot` 函数每次调用都返回了一个**新对象**，例如：

```js
function getSnapshot() {
  // 🔴 getSnapshot 不要总是返回不同的对象
  return {
    todos: myStore.todos
  };
}
```

如果 `getSnapshot` 返回值不同于上一次，React 会重新渲染组件。这就是为什么，如果总是返回一个不同的值，会进入到一个无限循环，并产生这个报错。

只有当确实有东西改变了，`getSnapshot` 才应该返回一个不同的对象。如果你的 store 包含不可变数据，可以直接返回此数据：

```js
function getSnapshot() {
  // ✅ 你可以返回不可变数据
  return myStore.todos;
}
```

如果你的 store 数据是可变的，`getSnapshot` 函数应当返回一个它的不可变快照。这意味着 **确实** 需要创建新对象，但不是每次调用都如此。而是应当保存最后一次计算得到的快照，并且在 store 中的数据不变的情况下，返回与上一次相同的快照。如何决定可变数据发生了改变则取决于你的可变 store。

### 我的 `subscribe` 函数每次重新渲染都被调用 

这里的 `subscribe` 函数是在组件 **内部** 定义的，所以它每次渲染都不同：

```js
function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  
  // 🚩 总是不同的函数，所以 React 每次重新渲染都会重新订阅
  function subscribe() {
    // ...
  }

  // ...
}
```

如果重新渲染时你传一个不同的 `subscribe` 函数，React 会重新订阅你的 store。如果这造成了性能问题，因而你想避免重新订阅，就把 `subscribe` 函数移到外面：

```js
function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  // ...
}

// ✅ 总是相同的函数，所以 React 不需要重新订阅
function subscribe() {
  // ...
}
```

或者，把 `subscribe` 包在 [`useCallback`](https://zh-hans.react.dev/reference/react/useCallback) 里面以便只在某些参数改变时重新订阅：

```js
function ChatIndicator({ userId }) {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  
  // ✅ 只要 userId 不变，都是同一个函数
  const subscribe = useCallback(() => {
    // ...
  }, [userId]);

  // ...
}
```

# useTransition

`useTransition` 是一个让你在不阻塞 UI 的情况下来更新状态的 React Hook。

```js
const [isPending, startTransition] = useTransition()
```

## 参考 

### `useTransition()` 

在组件的顶层调用 `useTransition`，将某些状态更新标记为转换状态。

```js
import { useTransition } from 'react';

function TabContainer() {
  const [isPending, startTransition] = useTransition();
  // ...
}
```

#### 参数 

`useTransition` 不需要任何参数。

#### 返回值 

`useTransition` 返回一个由两个元素组成的数组：

1. `isPending` 标志，告诉你是否存在**待处理的转换**。
2. [`startTransition` 函数](https://zh-hans.react.dev/reference/react/useTransition#starttransition) 允许你将状态更新**标记为转换状态**。

### `startTransition` 函数 

`useTransition` 返回的 `startTransition` 函数允许你将状态更新标记为转换状态。

```js
function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

#### 参数 

- 作用域（scope）：一个通过调用一个或多个 [`set` 函数](https://zh-hans.react.dev/reference/react/useState#setstate)。 函数更新某些状态的函数。React 会立即不带参数地调用此函数，并将在作用域函数调用期间计划同步执行的所有状态更新标记为转换状态。它们将是非阻塞的，并且 [不会显示不想要的加载指示器](https://zh-hans.react.dev/reference/react/useTransition#preventing-unwanted-loading-indicators)。

#### 返回值 

`startTransition` 不会返回任何值。

#### 注意事项 

- `useTransition` 是一个 Hook，因此只能在组件或自定义 Hook 内部调用。如果你需要在其他地方启动转换（例如从数据库），请调用独立的 [`startTransition`](https://zh-hans.react.dev/reference/react/startTransition) 函数。
- 只有在你可以访问该状态的 `set` 函数时，才能将更新包装为转换状态。如果你想响应某个 prop 或自定义 Hook 值启动转换，请尝试使用 [`useDeferredValue`](https://zh-hans.react.dev/reference/react/useDeferredValue)。
- 你传递给 `startTransition` 的函数必须是同步的。React 立即执行此函数，标记其执行期间发生的所有状态更新为转换状态。如果你稍后尝试执行更多的状态更新（例如在一个定时器中），它们将不会被标记为转换状态。
- 标记为转换状态的状态更新将被其他状态更新打断。例如，如果你在转换状态中更新图表组件，但在图表正在重新渲染时开始在输入框中输入，React 将在处理输入更新后重新启动对图表组件的渲染工作。
- 转换状态更新不能用于控制文本输入。
- 如果有多个正在进行的转换状态，React 目前会将它们批处理在一起。这是一个限制，可能会在未来的版本中被删除。

## 用法 

### 将状态更新标记为非阻塞转换状态 

在组件的顶层调用 `useTransition`，将状态更新标记为非阻塞的转换状态。

```js
import { useState, useTransition } from 'react';

function TabContainer() {
  const [isPending, startTransition] = useTransition();
  // ...
}
```

`useTransition` 返回一个具有两个项的数组：

1. **`isPending` 标志**，告诉你是否存在挂起的转换状态。
2. **`startTransition` 方法** 允许你将状态更新标记为转换状态。

你可以按照以下方式将状态更新标记为转换状态：

```js
function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

转换状态可以使用户界面更新在慢速设备上仍保持响应性。

通过转换状态，在重新渲染过程中你的用户界面保持响应。例如，如果用户单击一个选项卡，但改变了主意并单击另一个选项卡，他们可以在不等待第一个重新渲染完成的情况下完成操作。

#### useTransition 和 普通state更新的区别

##### 在转换状态中更新当前选项卡 

在此示例中，“文章”选项卡被 **人为地减慢**，以便至少需要一秒钟才能呈现。

单击“文章”，然后立即单击“联系人”。请注意，这会中断“文章”的缓慢渲染。 “联系人”选项卡立即显示。因为此状态更新被标记为转换状态，因此缓慢的重新渲染不会冻结用户界面。

<iframe src="https://codesandbox.io/embed/kind-pare-mffivk?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="kind-pare-mffivk"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

##### 在不使用转换状态的情况下更新当前选项卡 

在此示例中，“帖子”选项卡同样被 **人为地减慢**，以便至少需要一秒钟才能渲染。与之前的示例不同，这个状态更新 **不是一个转换状态**。

点击“帖子”，然后立即点击“联系人”。请注意，应用程序在渲染减速选项卡时会冻结，UI变得不响应。这个状态更新 **不是一个转换状态**，所以慢速的重新渲染会冻结用户界面。

<iframe src="https://codesandbox.io/embed/loving-ptolemy-ily1q3?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="loving-ptolemy-ily1q3"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 在转换中更新父组件 

你也可以通过 `useTransition` 调用来更新父组件的状态。例如，`TabButton` 组件在一个转换中包装了它的onClick逻辑：

```jsx
export default function TabButton({ children, isActive, onClick }) {
  const [isPending, startTransition] = useTransition();
  if (isActive) {
    return <b>{children}</b>
  }
  return (
    <button onClick={() => {
      startTransition(() => {
        onClick();
      });
    }}>
      {children}
    </button>
  );
}
```

因为父组件在 `onClick` 事件处理程序内更新了它的状态，所以该状态更新被标记为一个转换。这就是为什么，就像之前的例子一样，你可以单击“帖子”，然后立即单击“联系人”。更新选定选项卡被标记为一个转换，因此它不会阻止用户交互。

<iframe src="https://codesandbox.io/embed/sharp-scooby-pc8po3?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="sharp-scooby-pc8po3"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 在转换期间显示待处理的视觉状态 

你可以使用 `useTransition` 返回的 `isPending `布尔值来向用户指示转换正在进行中。例如，选项卡按钮可以有一个特殊的“待处理”视觉状态：

```js
function TabButton({ children, isActive, onClick }) {
  const [isPending, startTransition] = useTransition();
  // ...
  if (isPending) {
    return <b className="pending">{children}</b>;
  }
  // ...
```

请注意，现在单击“帖子”感觉更加灵敏，因为选项卡按钮本身立即更新了：

<iframe src="https://codesandbox.io/embed/lingering-forest-d667o1?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="lingering-forest-d667o1"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 避免不必要的加载指示器 

在这个例子中，`PostsTab` 组件使用启用了 [Suspense-enabled](https://zh-hans.react.dev/reference/react/Suspense) 的数据源获取一些数据。当你单击“帖子”选项卡时，`PostsTab` 组件将 **挂起**，导致最近的加载占位符出现：

<iframe src="https://codesandbox.io/embed/hungry-hellman-u75l9m?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="hungry-hellman-u75l9m"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

隐藏整个选项卡容器以显示加载指示符会导致用户体验不连贯。如果你将 `useTransition` 添加到 `TabButton` 中，你可以改为在选项卡按钮中指示待处理状态。

请注意，现在单击“帖子”不再用一个Suspense替换整个选项卡容器：

<iframe src="https://codesandbox.io/embed/crazy-sky-qj166n?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="crazy-sky-qj166n"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

[了解有关在Suspense中使用转换的更多信息](https://zh-hans.react.dev/reference/react/Suspense#preventing-already-revealed-content-from-hiding)。

> ### 注意
>
> 转换效果只会“等待”足够长的时间来避免隐藏 **已经显示** 的内容（例如选项卡容器）。如果“帖子”选项卡具有一个[嵌套 `<Suspense>` 边界](https://zh-hans.react.dev/reference/react/Suspense#revealing-nested-content-as-it-loads)，转换效果将不会“等待”它。

### 构建一个Suspense-enabled 的路由

如果你正在构建一个 React 框架或路由，我们建议将页面导航标记为转换效果。

```js
function Router() {
  const [page, setPage] = useState('/');
  const [isPending, startTransition] = useTransition();

  function navigate(url) {
    startTransition(() => {
      setPage(url);
    });
  }
  // ...
```

这么做有两个好处：

- [转换效果是可中断的](https://zh-hans.react.dev/reference/react/useTransition#marking-a-state-update-as-a-non-blocking-transition)，这样用户可以在等待重新渲染完成之前点击其他地方。
- [转换效果可以防止不必要的加载指示符](https://zh-hans.react.dev/reference/react/useTransition#preventing-unwanted-loading-indicators)，这样用户就可以避免在导航时产生不协调的跳转。

下面是一个简单的使用转换效果进行页面导航的路由器示例：

<iframe src="https://codesandbox.io/embed/frosty-scott-exderr?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="frosty-scott-exderr"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

> ### 注意
>
> [Suspense-enabled](https://zh-hans.react.dev/reference/react/Suspense) 的路由默认情况下会将页面导航更新包装成转换效果。

## 疑难解答 

### 在转换过程中更新输入无法正常工作 

你不能使用转换来控制输入的状态变量：

```jsx
const [text, setText] = useState('');
// ...
function handleChange(e) {
  // ❌ Can't use transitions for controlled input state
  startTransition(() => {
    setText(e.target.value);
  });
}
// ...
return <input value={text} onChange={handleChange} />;
```

这是因为转换是非阻塞的，但是在响应更改事件时更新输入应该是同步的。如果你想在输入时运行一个转换，有两个选项：

1. 你可以声明两个分开的状态变量：一个用于输入状态（它总是同步更新），另一个用于在转换中更新的状态变量。这样，你可以使用同步状态控制输入，并将转换状态变量（它将“滞后”于输入）传递给其余的渲染逻辑。
2. 或者，你可以有一个状态变量，并添加 [`useDeferredValue`](https://zh-hans.react.dev/reference/react/useDeferredValue)，它将“滞后”于实际值。它会自动触发非阻塞的重新渲染以“追赶”新值。

### React 没有将我的状态更新视为转换 

当你在转换中包装一个状态更新时，请确保它发生在 `startTransition` 调用期间：

```js
startTransition(() => {
  // ✅ Setting state *during* startTransition call
  setPage('/about');
});
```

传递给 `startTransition` 的函数必须是同步的。

你不能像这样将更新标记为转换：

```js
startTransition(() => {
  // ❌ Setting state *after* startTransition call
  setTimeout(() => {
    setPage('/about');
  }, 1000);
});
```

相反，你可以这样做：

```js
setTimeout(() => {
  startTransition(() => {
    // ✅ Setting state *during* startTransition call
    setPage('/about');
  });
}, 1000);
```

类似地，你不能像这样将更新标记为转换：

```js
startTransition(async () => {
  await someAsyncFunction();
  // ❌ Setting state *after* startTransition call
  setPage('/about');
});
```

然而，使用以下方法可以正常工作：

```js
await someAsyncFunction();
startTransition(() => {
  // ✅ Setting state *during* startTransition call
  setPage('/about');
});
```

### 我想在组件外部调用 `useTransition` 

你不能在组件外部调用 `useTransition`，因为它是一个 Hook。在这种情况下，请改用独立的 [`startTransition`](https://zh-hans.react.dev/reference/react/startTransition) 方法。它的工作方式相同，但不提供 `isPending` 指示器。

### 我传递给 `startTransition` 的函数会立即执行 

如果你运行这段代码，它将会打印 1, 2, 3：

```js
console.log(1);
startTransition(() => {
  console.log(2);
  setPage('/about');
});
console.log(3);
```

**期望打印 1, 2, 3**。 传递给 `startTransition` 的函数不会被延迟执行。与浏览器的 `setTimeout` 不同，它不会延迟执行回调。React 会立即执行你的函数，但是在它运行的同时安排的任何状态更新都被标记为转换。你可以将其想象为以下方式：

```js
// A simplified version of how React works

let isInsideTransition = false;

function startTransition(scope) {
  isInsideTransition = true;
  scope();
  isInsideTransition = false;
}

function setState() {
  if (isInsideTransition) {
    // ... schedule a transition state update ...
  } else {
    // ... schedule an urgent state update ...
  }
}
```

