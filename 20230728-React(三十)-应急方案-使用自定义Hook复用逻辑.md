# 20230728-React(三十)-应急方案-使用自定义Hook复用逻辑

React 有一些内置 Hook，例如 `useState`，`useContext` 和 `useEffect`。有时你需要一个用途更特殊的 Hook：例如获取数据，记录用户是否在线或者连接聊天室。虽然 React 中可能没有这些 Hook，但是你可以根据应用需求创建自己的 Hook。

> ### 你将会学习到
>
> - 什么是自定义 Hook，以及如何编写
> - 如何在组件间重用逻辑
> - 如何给自定义 Hook 命名以及如何构建
> - 提取自定义 Hook 的时机和原因

## 自定义 Hook：组件间共享逻辑 

假设你正在开发一款重度依赖网络的应用（和大多数应用一样）。当用户使用应用时网络意外断开，你需要提醒他。你会怎么处理呢？看上去组件需要两个东西：

1. 一个追踪网络是否在线的 state。
2. 一个订阅全局 [`online`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/online_event) 和 [`offline`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/offline_event) 事件并更新上述 state 的 Effect。

这会让组件与网络状态保持 [同步](https://zh-hans.react.dev/learn/synchronizing-with-effects)。你也许可以像这样开始：

<iframe src="https://codesandbox.io/embed/youthful-carson-nsny83?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="youthful-carson-nsny83"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

试着开启和关闭网络，注意观察 `StatusBar` 组件应对你的行为是如何更新的。

假设现在你想在另一个不同的组件里 **也** 使用同样的逻辑。你希望实现一个保存按钮，每当网络断开这个按钮就会不可用并且显示“Reconnecting…”而不是“Save progress”。

你可以从复制粘贴 `isOnline` state 和 Effect 到 `SaveButton` 组件开始：

<iframe src="https://codesandbox.io/embed/solitary-fast-b99g0g?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="solitary-fast-b99g0g"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

如果你关闭网络，可以发现这个按钮的外观变了。

这两个组件都能很好地工作，但不幸的是他们的逻辑重复了。他们看上去有不同的 **视觉外观**，但你依然想复用他们的逻辑。

### 从组件中提取自定义 Hook 

假设有一个内置 Hook `useOnlineStatus`，它与 [`useState`](https://zh-hans.react.dev/reference/react/useState) 和 [`useEffect`](https://zh-hans.react.dev/reference/react/useEffect) 相似。那么你就可以简化这两个组件并移除他们之间的重复部分：

```js
function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log('✅ Progress saved');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Save progress' : 'Reconnecting...'}
    </button>
  );
}
```

尽管目前还没有这样的内置 Hook，但是你可以自己写。声明一个 `useOnlineStatus` 函数，并把组件里早前写的所有重复代码移入该函数：

```js
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}
```

在函数结尾处返回 `isOnline`。这可以让组件读取到该值：

<iframe src="https://codesandbox.io/embed/gracious-chebyshev-qkpyz8?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="gracious-chebyshev-qkpyz8"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

切换网络状态验证一下是否会同时更新两个组件。

现在组件里没有那么多的重复逻辑了。**更重要的是，组件内部的代码描述的是想要做什么（使用在线状态！），而不是怎么做（通过订阅浏览器事件完成）**。

当提取逻辑到自定义 Hook 时，你可以隐藏如何处理外部系统或者浏览器 API 这些乱七八糟的细节。组件内部的代码表达的是目标而不是具体实现。

### Hook 的名称必须永远以 `use` 开头 

React 应用是由组件构成，而组件由内置或自定义 Hook 构成。可能你经常使用别人写的自定义 Hook，但偶尔也要自己写！

你必须遵循以下这些命名公约：

1. **React 组件名称必须以大写字母开头**，比如 `StatusBar` 和 `SaveButton`。React 组件还需要返回一些 React 能够显示的内容，比如一段 JSX。
2. **Hook 的名称必须以后跟一个大写字母的 `use` 开头**，像 [`useState`](https://zh-hans.react.dev/reference/react/useState)（内置） 或者 `useOnlineStatus`（像本文早前的自定义 Hook）。Hook 可以返回任意值。

这个公约保证你始终能一眼识别出组件并且知道它的 state，Effect 以及其他的 React 特性可能“隐藏”在哪里。例如如果你在组件内部看见 `getColor()` 函数调用，就可以确定它里面不可能包含 React state，因为它的名称没有以 `use` 开头。但是像 `useOnlineStatus()` 这样的函数调用就很可能包含对内部其他 Hook 的调用！

### 注意

如果你为 [React 配置了](https://zh-hans.react.dev/learn/editor-setup#linting) 代码检查工具，它会强制执行这个命名公约。现在滑动到上面的 sandbox，并将 `useOnlineStatus` 重命名为 `getOnlineStatus`。注意此时代码检查工具将不会再允许你在其内部调用 `useState` 或者 `useEffect`。只有 Hook 和组件可以调用其他 Hook！

### 深入探讨: 渲染期间调用的所有函数都应该以 use 前缀开头么？

不。没有 **调用** Hook 的函数不需要 **变成** Hook。

如果函数没有调用任何 Hook，请避免使用 `use` 前缀。 而是 **不带** `use` 前缀把它当成常规函数去写。例如下面的 `useSorted`  没有调用 Hook，所以叫它 `getSorted`：

```js
// 🔴 Avoid: 没有调用其他Hook的Hook
function useSorted(items) {
  return items.slice().sort();
}

// ✅ Good: 没有使用Hook的常规函数
function getSorted(items) {
  return items.slice().sort();
}
```

这保证你的代码可以在包含条件语句在内的任何地方调用这个常规函数：

```js
function List({ items, shouldSort }) {
  let displayedItems = items;
  if (shouldSort) {
    // ✅ 在条件分支里调用getSorted()是没问题的，因为它不是Hook
    displayedItems = getSorted(items);
  }
  // ...
}
```

哪怕内部只使用了一个 Hook，你也应该给这个函数加 `use` 前缀（让它成为一个 Hook）：

```js
// ✅ Good: 一个使用了其他Hook的Hook
function useAuth() {
  return useContext(Auth);
}
```

技术上 React 对此并不强制要求。原则上你可以写出不调用其他 Hook 的 Hook。但这常常会难以理解且受限，所以最好避免这种方式。但是它在极少数场景下可能是有益的。例如函数目前也许并没有使用任何 Hook，但是你计划未来在该函数内部添加一些 Hook 调用。那么使用 `use` 前缀命名就很有意义：

```js
// ✅ Good: 之后可能使用其他Hook的Hook
function useAuth() {
  // TODO: 当认证功能实现以后，替换这一行：
  // 返回 useContext(Auth)；
  return TEST_USER;
}
```

接下来组件就不能在条件语句里调用这个函数。当你在内部实际添加了 Hook 调用时，这一点将变得很重要。如果你（现在或者之后）没有计划在内部使用 Hook，请不要让它变成 Hook。

### 自定义 Hook 共享的是状态逻辑，而不是状态本身 

之前的例子里，当你开启或关闭网络时，两个组件一起更新了。但是两个组件共享 state 变量 `isOnline` 这种想法是错的。看这段代码：

```js
function StatusBar() {
  const isOnline = useOnlineStatus();
  // ...
}

function SaveButton() {
  const isOnline = useOnlineStatus();
  // ...
}
```

它的工作方式和你之前提取的重复代码一模一样：

```js
function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    // ...
  }, []);
  // ...
}

function SaveButton() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    // ...
  }, []);
  // ...
}
```

这是完全独立的两个 state 变量和 Effect！只是碰巧同一时间值一样，因为你使用了相同的外部值同步两个组件（无论网络是否开启）。

为了更好的说明这一点，我们需要一个不同的示例。看下面的 `Form` 组件：

<iframe src="https://codesandbox.io/embed/goofy-franklin-66tyej?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="goofy-franklin-66tyej"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

每个表单域都有一部分重复的逻辑：

1. 都有一个 state（`firstName` 和 `lastName`）。
2. 都有 change 事件的处理函数（`handleFirstNameChange` 和 `handleLastNameChange`）。
3. 都有为输入框指定 `value` 和 `onChange` 属性的 JSX。

你可以提取重复的逻辑到自定义 Hook `useFormInput`：

<iframe src="https://codesandbox.io/embed/sad-allen-bed9z6?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="sad-allen-bed9z6"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

注意它只声明了 **一个** state 变量，叫做 `value`。

但 `Form` 组件调用了 **两次** `useFormInput`：

```js
function Form() {
  const firstNameProps = useFormInput('Mary');
  const lastNameProps = useFormInput('Poppins');
  // ...

```

这就是为什么它工作的时候像声明了两个单独的 state 变量！

**自定义 Hook 共享的只是状态逻辑而不是状态本身。对 Hook 的每个调用完全独立于对同一个 Hook 的其他调用**。这就是上面两个 sandbox 结果完全相同的原因。如果愿意，你可以划上去进行比较。提取自定义 Hook 前后组件的行为是一致的。

当你需要在多个组件之间共享 state 本身时，需要 [将变量提升并传递下去](https://zh-hans.react.dev/learn/sharing-state-between-components)。

## 在 Hook 之间传递响应值 

每当组件重新渲染，自定义 Hook 中的代码就会重新运行。这就是组件和自定义 Hook 都 [需要是纯函数](https://zh-hans.react.dev/learn/keeping-components-pure) 的原因。我们应该把自定义 Hook 的代码看作组件主体的一部分。

由于自定义 Hook 会随着组件一起重新渲染，所以组件可以一直接收到最新的 props 和 state。想知道这意味着什么，那就看看这个聊天室的示例。修改 ServeUrl 或者 roomID：

<iframe src="https://codesandbox.io/embed/loving-blackwell-dpp3h9?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="loving-blackwell-dpp3h9"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

当你修改 `serverUrl` 或者 `roomId` 时，Effect 会对 [你的修改做出“响应”](https://zh-hans.react.dev/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values) 并重新同步。你可以通过每次修改 Effect 依赖项时聊天室重连的控制台消息来区分。

现在将 Effect 代码移入自定义 Hook：

```js
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      showNotification('New message: ' + msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

这让 `ChatRoom` 组件调用自定义 Hook，而不需要担心内部怎么工作：

```jsx
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });

  return (
    <>
      <label>
        Server URL:
        <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}
```

这看上去简洁多了（但是它做的是同一件事）！

注意逻辑 **仍然响应** props 和 state 的变化。尝试编辑 server URL 或选中的房间：

<iframe src="https://codesandbox.io/embed/elegant-dust-jqrn6q?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="elegant-dust-jqrn6q"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

注意你如何获取 Hook 的返回值：

```js
export default function ChatRoom({ roomId }) {
  //获取serverUrl
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });
  // ...
```

并把它作为输入传给另一个 Hook：

```js
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl //传入serverUrl
  });
  // ...
```

每次 `ChatRoom` 组件重新渲染，它就会传最新的 `roomId` 和 `serverUrl` 到你的 Hook。这就是每当重新渲染后他们的值不一样时你的 Effect 会重连聊天室的原因。（如果你曾经使用过音视频处理软件，像这样的 Hook 链也许会让你想起音视频效果链。好似 `useState` 的输出作为 `useChatRoom` 的输入）。

### 把事件处理函数传到自定义 Hook 中 

当你在更多组件中使用 `useChatRoom` 时，你可能希望组件能定制它的行为。例如现在 Hook 内部收到消息的处理逻辑是硬编码：

```js
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      showNotification('New message: ' + msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

假设你想把这个逻辑移回到组件中：

```js
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
    onReceiveMessage(msg) {
      showNotification('New message: ' + msg);
    }
  });
  // ...
```

完成这个工作需要修改自定义 Hook，把 `onReceiveMessage` 作为其命名选项之一：

```js
export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      onReceiveMessage(msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl, onReceiveMessage]); // ✅ 声明了所有的依赖
}
```

这个修改有效果，但是当自定义 Hook 接受事件处理函数时，你还可以进一步改进。

增加对 `onReceiveMessage` 的依赖并不理想，因为每次组件重新渲染时聊天室就会重新连接。 通过 [将这个事件处理函数包裹到 Effect Event 中来将它从依赖中移除](https://zh-hans.react.dev/learn/removing-effect-dependencies#wrapping-an-event-handler-from-the-props)：

```js
import { useEffect, useEffectEvent } from 'react';
// ...

export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  const onMessage = useEffectEvent(onReceiveMessage);

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      onMessage(msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ 声明所有依赖
}
```

现在每次 `ChatRoom` 组件重新渲染时聊天室都不会重连。这是一个将事件处理函数传给自定义 Hook 的完整且有效的 demo，你可以尝试一下：（只有修改room或者修改url才会触发重连，可以修改提示信息触发重新渲染，但是不会重连）

<iframe src="https://codesandbox.io/embed/frosty-bush-dlv6lr?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="frosty-bush-dlv6lr"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

注意你不再需要为了使用它而去了解 `useChatRoom` 是 **如何** 工作的。你可以把它添加到其他任意组件，传递其他任意选项，而它会以同样的方式工作。这就是自定义 Hook 的强大之处。

## 什么时候使用自定义 Hook 

你没必要对每段重复的代码都提取自定义 Hook。一些重复是好的。例如像早前提取的包裹单个 `useState` 调用的 `useFormInput` Hook 就是没有必要的。

但是每当你写 Effect 时，考虑一下把它包裹在自定义 Hook 是否更清晰。[你不应该经常使用 Effect](https://zh-hans.react.dev/learn/you-might-not-need-an-effect)，所以如果你正在写 Effect 就意味着你需要“走出 React”和某些外部系统同步，或者需要做一些 React 中没有对应内置 API 的事。把 Effect 包裹进自定义 Hook 可以准确表达你的目标以及数据在里面是如何流动的。

例如，假设 `ShippingForm` 组件展示两个下拉菜单：一个显示城市列表，另一个显示选中城市的区域列表。你可能一开始会像这样写代码：

```js
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  // 这个 Effect 拉取一个国家的城市数据
  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]);

  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
  // 这个 Effect 拉取选中城市的区域列表
  useEffect(() => {
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [city]);

  // ...
```

尽管这部分代码是重复的，但是 [把这些 Effect 各自分开是正确的](https://zh-hans.react.dev/learn/removing-effect-dependencies#is-your-effect-doing-several-unrelated-things)。他们同步两件不同的事情，所以不应该把他们合并到同一个 Effect。而是提取其中的通用逻辑到你自己的 `useData` Hook 来简化上面的 `ShippingForm` 组件：

```js
function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    if (url) {
      let ignore = false;
      fetch(url)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setData(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [url]);
  return data;
}
```

现在你可以在 `ShippingForm` 组件中调用 `useData` 替换两个 Effect：

```js
function ShippingForm({ country }) {
  const cities = useData(`/api/cities?country=${country}`);
  const [city, setCity] = useState(null);
  const areas = useData(city ? `/api/areas?city=${city}` : null);
  // ...
```

提取自定义 Hook 让数据流清晰。输入 `url`，就会输出 `data`。通过把 Effect “隐藏”在 `useData` 内部，你也可以防止一些正在处理 `ShippingForm` 组件的人向里面添加 [不必要的依赖](https://zh-hans.react.dev/learn/removing-effect-dependencies)。随着时间的推移，应用中大部分 Effect 都会存在于自定义 Hook 内部。

## 深入探讨: 让你的自定义 Hook 专注于具体的高级用例

从选择自定义 Hook 名称开始。如果你难以选择一个清晰的名称，这可能意味着你的 Effect 和组件逻辑剩余的部分耦合度太高，还没有做好被提取的准备。

理想情况下，你的自定义 Hook 名称应该清晰到即使一个不经常写代码的人也能很好地猜中自定义 Hook 的功能，输入和返回：

- ✅ `useData(url)`
- ✅ `useImpressionLog(eventName, extraData)`
- ✅ `useChatRoom(options)`

当你和外部系统同步的时候，你的自定义 Hook 名称可能会更加专业，并使用该系统特定的术语。只要对熟悉这个系统的人来说名称清晰就可以：

- ✅ `useMediaQuery(query)`
- ✅ `useSocket(url)`
- ✅ `useIntersectionObserver(ref, options)`

**保持自定义 Hook 专注于具体的高级用例**。避免创建和使用作为 `useEffect` API 本身的替代品和 wrapper 的自定义“生命周期” Hook：

- 🔴 `useMount(fn)`
- 🔴 `useEffectOnce(fn)`
- 🔴 `useUpdateEffect(fn)`

例如这个 `useMount` Hook 试图保证一些代码只在“加载”时运行：

```js
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  // 🔴 Avoid: 使用自定义“生命周期” Hook
  useMount(() => {
    const connection = createConnection({ roomId, serverUrl });
    connection.connect();

    post('/analytics/event', { eventName: 'visit_chat' });
  });
  // ...
}

// 🔴 Avoid: 创建自定义“生命周期” Hook
function useMount(fn) {
  useEffect(() => {
    fn();
  }, []); // 🔴 React Hook useEffect 缺少依赖项: 'fn'
}
```

**像 `useMount` 这样的自定义“生命周期” Hook 不是很适合 React 范式**。例如示例代码有一个错误（它没有对 `roomId` 或 `serverUrl` 的变化做出“响应” ），但是代码检查工具并不会向你发出对应的警告，因为它只能检测 `useEffect` 的直接调用。并不了解你的 Hook。

如果你正在编写 Effect，请从直接使用 React API 开始：

```js
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  // ✅ Good: 通过用途分割的两个原始Effect

  useEffect(() => {
    const connection = createConnection({ serverUrl, roomId });
    connection.connect();
    return () => connection.disconnect();
  }, [serverUrl, roomId]);

  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_chat', roomId });
  }, [roomId]);

  // ...
}
```

然后你可以（但不是必须的）为不同的高级用例提取自定义 Hook：

```js
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  // ✅ Great: 以用途命名的自定义Hook
  useChatRoom({ serverUrl, roomId });
  useImpressionLog('visit_chat', { roomId });
  // ...
}
```

**好的自定义 Hook 通过限制功能使代码调用更具声明性**。例如 `useChatRoom(options)` 只能连接聊天室，而 `useImpressionLog(eventName, extraData)` 只能向分析系统发送展示日志。如果你的自定义 Hook API 没有约束用例且非常抽象，那么在长期的运行中，它引入的问题可能比解决的问题更多。

### 自定义 Hook 帮助你迁移到更好的模式 

Effect 是一个 [“逃脱方案”](https://zh-hans.react.dev/learn/escape-hatches)：当需要“走出 React”且用例没有更好的内置解决方案时你可以使用他们。随着时间的推移，React 团队的目标是通过给更具体的问题提供更具体的解决方案来最小化应用中的 Effect 数量。把你的 Effect 包裹进自定义 Hook，当这些解决方案可用时升级代码会更加容易。

让我们回到这个示例：

<iframe src="https://codesandbox.io/embed/peaceful-leftpad-9x38on?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="peaceful-leftpad-9x38on"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

在上述示例中，`useOnlineStatus` 借助一组 [`useState`](https://zh-hans.react.dev/reference/react/useState) 和 [`useEffect`](https://zh-hans.react.dev/reference/react/useEffect) 实现。但这不是最好的解决方案。它有许多边界用例没有考虑到。例如假设当组件加载时，`isOnline` 已经为 `true`，但是如果网络已经离线的话这就是错误的。你可以使用浏览器的 [`navigator.onLine`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/onLine) API 来检查，但是在生成初始 HTML 的服务端直接使用它是没用的。简而言之这段代码可以改进。

幸运的是，React 18 包含了一个叫做 [`useSyncExternalStore`](https://zh-hans.react.dev/reference/react/useSyncExternalStore) 的专用 API，它可以解决你所有这些问题。这里展示了如何利用这个新 API 来重写你的 `useOnlineStatus` Hook：

<iframe src="https://codesandbox.io/embed/distracted-nightingale-z43tyt?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="distracted-nightingale-z43tyt"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

注意 **你不需要修改任何组件** 就能完成这次迁移：

```js
function StatusBar() {
  const isOnline = useOnlineStatus();
  // ...
}

function SaveButton() {
  const isOnline = useOnlineStatus();
  // ...
}
```

这是把 Effect 包裹进自定义 Hook 有益的另一个原因：

1. 你让进出 Effect 的数据流非常清晰。
2. 你让组件专注于目标，而不是 Effect 的准确实现。
3. 当 React 增加新特性时，你可以在不修改任何组件的情况下移除这些 Effect。

和 [设计系统](https://uxdesign.cc/everything-you-need-to-know-about-design-systems-54b109851969) 相似，你可能会发现从应用的组件中提取通用逻辑到自定义 Hook 是非常有帮助的。这会让你的组件代码专注于目标，并且避免经常写原始 Effect。许多很棒的自定义 Hook 是由 React 社区维护的。

## 深入探讨: React 会为数据获取提供内置解决方案么？

我们仍然在规划细节，但是期望未来可以像这样写数据获取：

```js
import { use } from 'react'; // 还不可用！

function ShippingForm({ country }) {
  const cities = use(fetch(`/api/cities?country=${country}`));
  const [city, setCity] = useState(null);
  const areas = city ? use(fetch(`/api/areas?city=${city}`)) : null;
  // ...
```

比起在每个组件手动写原始 Effect，在应用中使用像上面 `useData` 这样的自定义 Hook，之后迁移到最终推荐方式你所需要的修改更少。但是旧的方式仍然可以有效工作，所以如果你喜欢写原始 Effect，可以继续这样做。

### 不止一个方法可以做到 

假设你想要使用浏览器的 [`requestAnimationFrame`](https://developer.mozilla.org/zh-CN/docs/Web/API/window/requestAnimationFrame) API **从头开始** 实现一个 fade-in 动画。你也许会从一个设置动画循环的 Effect 开始。在动画的每一帧中，你可以修改 [ref 持有的](https://zh-hans.react.dev/learn/manipulating-the-dom-with-refs) DOM 节点的 opacity 属性直到 `1`。你的代码一开始可能是这样：

<iframe src="https://codesandbox.io/embed/eloquent-goldstine-9nf6pc?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="eloquent-goldstine-9nf6pc"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

为了让组件更具有可读性，你可能要将逻辑提取到自定义 Hook `useFadeIn`：

<iframe src="https://codesandbox.io/embed/quizzical-sunset-x7fkq8?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="quizzical-sunset-x7fkq8"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

你可以让 `useFadeIn` 和原来保持一致，但是也可以进一步重构。例如你可以把设置动画循环的逻辑从 `useFadeIn` 提取到自定义 Hook `useAnimationLoop`：

<iframe src="https://codesandbox.io/embed/condescending-merkle-w6nwnb?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="condescending-merkle-w6nwnb"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

但是 **没有必要** 这样做。和常规函数一样，最终是由你决定在哪里绘制代码不同部分之间的边界。你也可以采取不一样的方法。把大部分必要的逻辑移入一个 [JavaScript 类](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes)，而不是把逻辑保留在 Effect 中：

<iframe src="https://codesandbox.io/embed/summer-sound-2tf7xm?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="summer-sound-2tf7xm"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

Effect 可以连接 React 和外部系统。Effect 之间的配合越多（例如链接多个动画），像上面的 sandbox 一样 **完整地** 从 Effect 和 Hook 中提取逻辑就越有意义。然后你提取的代码 **变成** “外部系统”。这会让你的 Effect 保持简洁，因为他们只需要向已经被你移动到 React 外部的系统发送消息。

上面这个示例假设需要使用 JavaScript 写 fade-in 逻辑。但使用纯 [CSS 动画](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Animations/Using_CSS_animations) 实现这个特定的 fade-in 动画会更加简单和高效：

<iframe src="https://codesandbox.io/embed/epic-leftpad-krfjws?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="epic-leftpad-krfjws"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

某些时候你甚至不需要 Hook！

## 摘要

- 自定义 Hook 让你可以在组件间共享逻辑。
- 自定义 Hook 命名必须以后跟一个大写字母的 `use` 开头。
- 自定义 Hook 共享的只是状态逻辑，不是状态本身。
- 你可以将响应值从一个 Hook 传到另一个，并且他们会保持最新。
- 每次组件重新渲染时，所有的 Hook 会重新运行。
- 自定义 Hook 的代码应该和组件代码一样保持纯粹。
- 把自定义 Hook 收到的事件处理函数包裹到 Effect Event。
- 不要创建像 `useMount` 这样的自定义 Hook。保持目标具体化。
- 如何以及在哪里选择代码边界取决于你。

## 挑战

### 修复计时器重置

这个示例有 **两个** 独立的计时器。

`App` 组件调用 `useCounter`，这个 Hook 调用 `useInterval` 来每秒更新一次计数器。但是 `App` 组件 **也** 调用 `useInterval` 每两秒随机更新一次页面背景色。

更新页面背景色的回调函数因为一些原因从未执行过。在 `useInterval` 内部添加一些日志。

```js
useEffect(() => {
    console.log('✅ Setting up an interval with delay ', delay)
    const id = setInterval(onTick, delay);
    return () => {
      console.log('❌ Clearing an interval with delay ', delay)
      clearInterval(id);
    };
  }, [onTick, delay]);
```

这些日志符合你的预期吗？如果一些不必要的 Effect 似乎重新同步了，你能猜出哪一个依赖项导致了这个情况吗？有其他方式从 Effect 中 [移除依赖](https://zh-hans.react.dev/learn/removing-effect-dependencies) 吗？

这个问题修复以后，你预期的应该是页面背景每两秒更新一次。

<iframe src="https://codesandbox.io/embed/eager-maria-d4qr2v?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="eager-maria-d4qr2v"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

#### 提示

看上去你的 `useInterval` Hook 接受事件监听器作为参数。你能想到一些包裹事件监听器的方法，让它不需要成为 Effect 的依赖项吗？

#### 答案

和 [早前这个页面](https://zh-hans.react.dev/learn/reusing-logic-with-custom-hooks#passing-event-handlers-to-custom-hooks) 做的一样，在 `useInterval` 内部把 tick 回调函数包裹进一个 Effect Event。

这将让你可以从 Effect 的依赖项中删掉 `onTick`。每次组件重新渲染时，Effect 将不会重新同步，所以页面背景颜色变化 interval 有机会触发之前不会每秒重置一次。

随着这个修改，两个 interval 都会像预期一样工作并且不会互相干扰：

<iframe src="https://codesandbox.io/embed/competent-resonance-6ezr0m?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="competent-resonance-6ezr0m"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 实现交错运动 

这个示例中，`usePointerPosition()` Hook 追踪当前指针位置。尝试移动光标或你的手指到预览区域上方，可以看到有一个红点随着你移动。它的位置被保存在变量 `pos1` 中。

事实上，有 5（!）个正在被渲染的不同红点。你看不见是因为他们现在都显示在同一位置。这就是你需要修复的问题。你想要实现的是一个“交错”运动：每个圆点应该“跟随”它前一个点的路径。例如如果你快速移动光标，第一个点应该立刻跟着它，第二个应该在小小的延时后跟上第一个点，第三个点应该跟着第二个点等等。

你需要实现自定义 Hook `useDelayedValue`。它当前的实现返回的是提供给它的 `value`。而你想从 `delay` 毫秒之前返回 `value`。你可能需要一些 state 和一个 Effect 来完成这个任务。

实现 `useDelayedValue` 后，你应该看见这些点一个接一个运动。

<iframe src="https://codesandbox.io/embed/flamboyant-sid-mp5jl8?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="flamboyant-sid-mp5jl8"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

#### 提示

你需要在自定义 Hook 内部存储一个 state 变量 `delayedValue`。当 `value` 变化时，你需要运行一个 Effect。这个 Effect 应该在 `delay` 毫秒后更新 `delayedValue`。你可能发现调用 `setTimeout` 很有帮助。

这个 Effect 需要清理吗？为什么？

#### 答案

这里是一个生效的版本。你将 `delayedValue` 保存为一个 state 变量。当 `value` 更新时，Effect 会安排一个 timeout 来更新 `delayedValue`。这就是 `delayedValue` 总是“滞后于”实际 `value` 的原因。

<iframe src="https://codesandbox.io/embed/flamboyant-shape-9c2qek?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="flamboyant-shape-9c2qek"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

注意这个 Effect **不** 需要清理。如果你在清理函数中调用了 `clearTimeout`，那么每次 `value` 变化时，就会终止已经计划好的 timeout。为了保持运动连续，你需要触发所有 timeout。

## 个人总结useEffectEvent

> **个人总结**：useEffectEvent的作用更多是把一些**不需要响应式，但同时useEffect又必须要求声明为依赖的函数逻辑**，把它分离出去。分离到`useEffectEvent`中，通过useEffectEvent生成返回的函数，React认为它是**合法的不需要声明为依赖的函数**，所以可以直接在`useEffect`里使用。

