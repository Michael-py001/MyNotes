# 20230725-React(二十七)-应急方案-响应式 Effect 的生命周期

Effect 与组件有不同的生命周期。组件可以挂载、更新或卸载。Effect 只能做两件事：开始同步某些东西，然后停止同步它。如果 Effect 依赖于随时间变化的 props 和 state，这个循环可能会发生多次。React 提供了代码检查规则来检查是否正确地指定了 Effect 的依赖项，这能够使 Effect 与最新的 props 和 state 保持同步。

> ### 你将会学习到
>
> - Effect 的生命周期与组件的生命周期有何不同
> - 如何独立地考虑每个 Effect
> - 什么时候以及为什么 Effect 需要重新同步
> - 如何确定 Effect 的依赖项
> - 值是响应式的含义是什么
> - 空依赖数组意味着什么
> - React 如何使用检查工具验证依赖关系是否正确
> - 与代码检查工具产生分歧时，该如何处理

## Effect 的生命周期 

每个 React 组件都经历相同的生命周期：

- 当组件被添加到屏幕上时，它会进行组件的 **挂载**。
- 当组件接收到新的 props 或 state 时，通常是作为对交互的响应，它会进行组件的 **更新**。
- 当组件从屏幕上移除时，它会进行组件的 **卸载**。

**这是一种很好的思考组件的方式，但并不适用于 Effect**。相反，尝试从组件生命周期中跳脱出来，独立思考 Effect。Effect 描述了如何将外部系统与当前的 props 和 state 同步。随着代码的变化，同步的频率可能会增加或减少。

为了说明这一点，考虑下面这个示例。Effect 将组件连接到聊天服务器：

```js
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

Effect 的主体部分指定了如何 **开始同步**：

```js
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    // ...
```

Effect 返回的清理函数指定了如何 **停止同步**：

```js
 	// ...
    return () => {
      connection.disconnect();
    };
    // ...
```

你可能会直观地认为当组件挂载时 React 会 **开始同步**，而当组件卸载时会 **停止同步**。然而，事情并没有这么简单！有时，在组件保持挂载状态的同时，可能还需要 **多次开始和停止同步**。

让我们来看看 **为什么** 这是必要的、**何时** 会发生以及 **如何** 控制这种行为。

> ### 注意
>
> 有些 Effect 根本不返回清理函数。[在大多数情况下](https://zh-hans.react.dev/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)，可能希望返回一个清理函数，但如果没有返回，React 将表现得好像返回了一个空的清理函数。

### 为什么同步可能需要多次进行 

想象一下，这个 `ChatRoom` 组件接收 `roomId` 属性，用户可以在下拉菜单中选择。假设初始时，用户选择了 `"general"` 作为 `roomId`。应用程序会显示 `"general"` 聊天室：

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId /* "general" */ }) {
  // ...
  return <h1>欢迎来到 {roomId} 房间！</h1>;
}
```

在 UI 显示之后，React 将运行 Effect 来 **开始同步**。它连接到 `"general"` 聊天室：

```jsx
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 连接到 "general" 聊天室
    connection.connect();
    return () => {
      connection.disconnect(); // 断开与 "general" 聊天室的连接
    };
  }, [roomId]);
  // ...
```

到目前为止，一切都很顺利。

之后，用户在下拉菜单中选择了不同的房间（例如 `"travel"` ）。首先，React 会更新 UI：

```jsx
function ChatRoom({ roomId /* "travel" */ }) {
  // ...
  return <h1>欢迎来到 {roomId} 房间！</h1>;
}
```

思考接下来应该发生什么。用户在界面中看到 `"travel"` 是当前选定的聊天室。然而，上次运行的 Effect 仍然连接到 `"general"` 聊天室。**`roomId` 属性已经发生了变化，所以之前 Effect 所做的事情（连接到 `"general"` 聊天室）不再与 UI 匹配**。

此时，你希望 React 执行两个操作：

1. 停止与旧的 `roomId` 同步（断开与 `"general"` 聊天室的连接）
2. 开始与新的 `roomId` 同步（连接到 `"travel"` 聊天室）

**幸运的是，你已经教会了 React 如何执行这两个操作**！Effect 的主体部分指定了如何开始同步，而清理函数指定了如何停止同步。现在，React 只需要按照正确的顺序和正确的 props 和 state 来调用它们。让我们看看具体是如何实现的。

### React 如何重新同步 Effect 

回想一下，`ChatRoom` 组件已经接收到了 `roomId` 属性的新值。之前它是 `"general"`，现在变成了 `"travel"`。React 需要重新同步 Effect，以重新连接到不同的聊天室。

为了 **停止同步**，React 将调用 Effect 返回的清理函数，该函数在连接到 `"general"` 聊天室后返回。由于 `roomId` 为 `"general"`，清理函数将断开与 `"general"` 聊天室的连接：

```jsx
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 连接到 "general" 聊天室
    connection.connect();
    return () => {
      connection.disconnect(); // 断开与 "general" 聊天室的连接
    };
    // ...
```

然后，React 将运行在此渲染期间提供的 Effect。这次，`roomId` 为 `"travel"`，因此它将 **开始同步** 到 `"travel"` 聊天室（直到最终也调用了清理函数）：

```jsx
function ChatRoom({ roomId /* "travel" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 连接到 "travel" 聊天室
    connection.connect();
    // ...
```

多亏了这一点，现在已经连接到了用户在 UI 中选择的同一个聊天室。避免了灾难！

每当组件使用不同的 `roomId` 重新渲染后，Effect 将重新进行同步。例如，假设用户将 `roomId` 从 `"travel"` 更改为 `"music"`。React 将再次通过调用清理函数 **停止同步** Effect（断开与 `"travel"` 聊天室的连接）。然后，它将通过使用新的 `roomId` 属性再次运行 Effect 的主体部分 **开始同步**（连接到 `"music"` 聊天室）。

最后，当用户切换到不同的屏幕时，`ChatRoom` 组件将被卸载。现在没有必要保持连接了。React 将 **最后一次停止同步** Effect，并从 `"music"` 聊天室断开连接。

### 从 Effect 的角度思考 

让我们总结一下从 `ChatRoom` 组件的角度所发生的一切：

1. `ChatRoom` 组件挂载，`roomId` 设置为 `"general"`
2. `ChatRoom` 组件更新，`roomId` 设置为 `"travel"`
3. `ChatRoom` 组件更新，`roomId` 设置为 `"music"`
4. `ChatRoom` 组件卸载

在组件生命周期的每个阶段，Effect 执行了不同的操作：

1. Effect 连接到了 `"general"` 聊天室
2. Effect 断开了与 `"general"` 聊天室的连接，并连接到了 `"travel"` 聊天室
3. Effect 断开了与 `"travel"` 聊天室的连接，并连接到了 `"music"` 聊天室
4. Effect 断开了与 `"music"` 聊天室的连接

现在让我们从 Effect 本身的角度来思考所发生的事情：

```jsx
  useEffect(() => {
    // Effect 连接到了通过 roomId 指定的聊天室...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      // ...直到它断开连接
      connection.disconnect();
    };
  }, [roomId]);
```

这段代码的结构可能会将所发生的事情看作是一系列不重叠的时间段：

1. Effect 连接到了 `"general"` 聊天室（直到断开连接）
2. Effect 连接到了 `"travel"` 聊天室（直到断开连接）
3. Effect 连接到了 `"music"` 聊天室（直到断开连接）

之前，你是从组件的角度思考的。当你从组件的角度思考时，很容易将 Effect 视为在特定时间点触发的“回调函数”或“生命周期事件”，例如“渲染后”或“卸载前”。这种思维方式很快变得复杂，所以最好避免使用。

**相反，始终专注于单个启动/停止周期。无论组件是挂载、更新还是卸载，都不应该有影响。只需要描述如何开始同步和如何停止。如果做得好，Effect 将能够在需要时始终具备启动和停止的弹性**。

这可能会让你想起当编写创建 JSX 的渲染逻辑时，并不考虑组件是挂载还是更新。描述的是应该显示在屏幕上的内容，而 React 会 [解决其余的问题](https://zh-hans.react.dev/learn/reacting-to-input-with-state)。

### React 如何验证 Effect 可以重新进行同步 

这里有一个可以互动的实时示例。点击“打开聊天”来挂载 `ChatRoom` 组件：

<iframe src="https://codesandbox.io/embed/great-chandrasekhar-s7j9gj?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="great-chandrasekhar-s7j9gj"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

请注意，当组件首次挂载时，会看到三个日志：

1. `✅ 连接到 "general" 聊天室，位于 https://localhost:1234...` *(仅限开发环境)*
2. `❌ 从 "general" 聊天室断开连接，位于 https://localhost:1234.` *(仅限开发环境)*
3. `✅ 连接到 "general" 聊天室，位于 https://localhost:1234...`

前两个日志仅适用于开发环境。在开发环境中，React 总是会重新挂载每个组件一次。

**React 通过在开发环境中立即强制 Effect 重新进行同步来验证其是否能够重新同步**。这可能让你想起打开门并额外关闭它以检查门锁是否有效的情景。React 在开发环境中额外启动和停止 Effect 一次，以检查 [是否正确实现了它的清理功能](https://zh-hans.react.dev/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)。

实际上，Effect 重新进行同步的主要原因是它所使用的某些数据发生了变化。在上面的示例中，更改所选的聊天室。注意当 `roomId` 发生变化时，Effect 会重新进行同步。

然而，还存在其他一些不寻常的情况需要重新进行同步。例如，在上面的示例中，尝试在聊天打开时编辑 `serverUrl`。注意当修改代码时，Effect会重新进行同步。将来，React 可能会添加更多依赖于重新同步的功能。

### React 如何知道需要重新进行 Effect 的同步 

你可能想知道 React 是如何知道在 `roomId` 更改后需要重新同步 Effect。这是因为 **你告诉了 React** 它的代码依赖于 `roomId`，通过将其包含在 [依赖列表](https://zh-hans.react.dev/learn/synchronizing-with-effects#step-2-specify-the-effect-dependencies) 中。

```js
function ChatRoom({ roomId }) { // roomId 属性可能会随时间变化。
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 这个 Effect 读取了 roomId
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]); // 因此，你告诉 React 这个 Effect "依赖于" roomId
  // ...
```

下面是它的工作原理：

1. 你知道 `roomId` 是 prop，这意味着它可能会随着时间的推移发生变化。
2. 你知道 Effect 读取了 `roomId`（因此其逻辑依赖于可能会在之后发生变化的值）。
3. 这就是为什么你将其指定为 Effect 的依赖项（以便在 `roomId` 发生变化时重新进行同步）。

每次在组件重新渲染后，React 都会查看传递的依赖项数组。如果数组中的任何值与上一次渲染时在相同位置传递的值不同，React 将重新同步 Effect。

例如，如果在初始渲染时传递了 `["general"]`，然后在下一次渲染时传递了 `["travel"]`，React 将比较 `"general"` 和 `"travel"`。这些是不同的值（使用 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 进行比较），因此 React 将重新同步 Effect。另一方面，如果组件重新渲染但 `roomId` 没有发生变化，Effect 将继续连接到相同的房间。

### 每个 Effect 表示一个独立的同步过程。 

抵制将与 Effect 无关的逻辑添加到已经编写的 Effect 中，仅仅因为这些逻辑需要与 Effect 同时运行。例如，假设你想在用户访问房间时发送一个分析事件。你已经有一个依赖于 `roomId` 的 Effect，所以你可能会想要将分析调用添加到那里：

```js
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

但是想象一下，如果以后给这个 Effect 添加了另一个需要重新建立连接的依赖项。如果这个 Effect 重新进行同步，它将为相同的房间调用 `logVisit(roomId)`，而这不是你的意图。记录访问行为是 **一个独立的过程**，与连接不同。将它们作为两个单独的 Effect 编写：

```js
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);
  }, [roomId]);

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    // ...
  }, [roomId]);
  // ...
}
```

**代码中的每个 Effect 应该代表一个独立的同步过程。.**

在上面的示例中，删除一个 Effect 不会影响另一个 Effect 的逻辑。这表明它们同步不同的内容，因此将它们拆分开是有意义的。另一方面，如果将一个内聚的逻辑拆分成多个独立的 Effects，代码可能会看起来更加“清晰”，但 [维护起来会更加困难](https://zh-hans.react.dev/learn/you-might-not-need-an-effect#chains-of-computations)。这就是为什么你应该考虑这些过程是相同还是独立的，而不是只考虑代码是否看起来更整洁。

## Effect 会“响应”于响应式值 

Effect 读取了两个变量（`serverUrl` 和 `roomId`），但是只将 `roomId` 指定为依赖项：

```js
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

为什么 `serverUrl` 不需要作为依赖项呢？

这是因为 `serverUrl` 永远不会因为重新渲染而发生变化。无论组件重新渲染多少次以及原因是什么，`serverUrl` 都保持不变。既然 `serverUrl` 从不变化，将其指定为依赖项就没有意义。毕竟，依赖项只有在随时间变化时才会起作用！

另一方面，`roomId` 在重新渲染时可能会不同。**在组件内部声明的 props、state 和其他值都是 响应式 的，因为它们是在渲染过程中计算的，并参与了 React 的数据流**。

如果 `serverUrl` 是状态变量，那么它就是响应式的。响应式值必须包含在依赖项中：

```js
function ChatRoom({ roomId }) { // Props 随时间变化
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // State 可能随时间变化

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Effect 读取 props 和 state
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // 因此，你告诉 React 这个 Effect "依赖于" props 和 state
  // ...
}
```

通过将 `serverUrl` 包含在依赖项中，确保 Effect 在其发生变化后重新同步。

尝试在此沙盒中更改所选的聊天室或编辑服务器 URL：

<iframe src="https://codesandbox.io/embed/sleepy-cherry-4ltw8v?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="sleepy-cherry-4ltw8v"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

无论何时更改一个类似 `roomId` 或 `serverUrl` 的响应式值，该 Effect 都会重新连接到聊天服务器。

### 没有依赖项的 Effect 的含义 

如果将 `serverUrl` 和 `roomId` 都移出组件会发生什么？

```js
const serverUrl = 'https://localhost:1234';
const roomId = 'general';

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ 声明的所有依赖
  // ...
}
```

现在 Effect 的代码不使用任何响应式值，因此它的依赖可以是空的 (`[]`)。

从组件的角度来看，空的 `[]` 依赖数组意味着这个 Effect 仅在组件挂载时连接到聊天室，并在组件卸载时断开连接。（请记住，在开发环境中，React 仍会 [额外执行一次](https://zh-hans.react.dev/learn/lifecycle-of-reactive-effects#how-react-verifies-that-your-effect-can-re-synchronize) 来对逻辑进行压力测试。）

<iframe src="https://codesandbox.io/embed/morning-moon-84vkdt?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="morning-moon-84vkdt"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

然而，如果你 [从 Effect 的角度思考](https://zh-hans.react.dev/learn/lifecycle-of-reactive-effects#thinking-from-the-effects-perspective)，根本不需要考虑挂载和卸载。重要的是，你已经指定了 Effect 如何开始和停止同步。目前，它没有任何响应式依赖。但是，如果希望用户随时间改变 `roomId` 或 `serverUrl`（它们将变为响应式），Effect 的代码不需要改变。只需要将它们添加到依赖项中即可。

### 在组件主体中声明的所有变量都是响应式的 

Props 和 state 并不是唯一的响应式值。从它们计算出的值也是响应式的。如果 props 或 state 发生变化，组件将重新渲染，从中计算出的值也会随之改变。这就是为什么 Effect 使用的组件主体中的所有变量都应该在依赖列表中。

假设用户可以在下拉菜单中选择聊天服务器，但他们还可以在设置中配置默认服务器。假设你已经将设置状态放入了 [上下文](https://zh-hans.react.dev/learn/scaling-up-with-reducer-and-context)，因此从该上下文中读取 `settings`。现在，可以根据 props 中选择的服务器和默认服务器来计算 `serverUrl`：

```js
function ChatRoom({ roomId, selectedServerUrl }) { // roomId 是响应式的
  const settings = useContext(SettingsContext); // settings 是响应式的
  const serverUrl = selectedServerUrl ?? settings.defaultServerUrl; // serverUrl 是响应式的
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Effect 读取了 roomId 和 serverUrl
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // 因此，当它们中的任何一个发生变化时，它需要重新同步！
  // ...
}
```

在这个例子中，`serverUrl` 不是 prop 或 state 变量。它是在渲染过程中计算的普通变量。但是它是在渲染过程中计算的，所以它可能会因为重新渲染而改变。这就是为什么它是响应式的。

**组件内部的所有值（包括 props、state 和组件体内的变量）都是响应式的。任何响应式值都可以在重新渲染时发生变化，所以需要将响应式值包括在 Effect 的依赖项中**。

换句话说，Effect 对组件体内的所有值都会“react”。

## 深入探讨: 全局变量或可变值可以作为依赖项吗？ 

可变值（包括全局变量）不是响应式的。

例如，像 [`location.pathname`](https://developer.mozilla.org/zh-CN/docs/Web/API/Location/pathname) 这样的可变值不能作为依赖项。它是可变的，因此可以在 React 渲染数据流之外的任何时间发生变化。更改它不会触发组件的重新渲染。因此，即使在依赖项中指定了它，React 也无法知道在其更改时重新同步 Effect。这也违反了 React 的规则，因为在渲染过程中读取可变数据（即在计算依赖项时）会破坏 [纯粹的渲染](https://zh-hans.react.dev/learn/keeping-components-pure)。相反，应该使用 [`useSyncExternalStore`](https://zh-hans.react.dev/learn/you-might-not-need-an-effect#subscribing-to-an-external-store) 来读取和订阅外部可变值。

**另外，像 [`ref.current`](https://zh-hans.react.dev/reference/react/useRef#reference) 或从中读取的值也不能作为依赖项。`useRef` 返回的 ref 对象本身可以作为依赖项**，但其 `current` 属性是有意可变的。它允许 [跟踪某些值而不触发重新渲染](https://zh-hans.react.dev/learn/referencing-values-with-refs)。但由于更改它不会触发重新渲染，它不是响应式值，React 不会知道在其更改时重新运行 Effect。

正如你将在本页面下面学到的那样，检查工具将自动检查这些问题。

### React 会验证是否将每个响应式值都指定为了依赖项 

如果检查工具 [配置了 React](https://zh-hans.react.dev/learn/editor-setup#linting)，它将检查 Effect 代码中使用的每个响应式值是否已声明为其依赖项。例如，以下示例是一个 lint 错误，因为 `roomId` 和 `serverUrl` 都是响应式的：

<iframe src="https://codesandbox.io/embed/keen-margulis-6ymg94?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="keen-margulis-6ymg94"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

> Lint Error: 
>
> ```
> 11:6 - React Hook useEffect has missing dependencies: 'roomId' and 'serverUrl'. Either include them or remove the dependency array.
> ```

这可能看起来像是 React 错误，但实际上 React 是在指出代码中的 bug。`roomId` 和 `serverUrl` 都可能随时间改变，但忘记了在它们改变时重新同步 Effect。即使用户在 UI 中选择了不同的值，仍然保持连接到初始的 `roomId` 和 `serverUrl`。

要修复这个 bug，请按照检查工具的建议将 `roomId` 和 `serverUrl` 作为 Effect 的依赖进行指定：

```js
function ChatRoom({ roomId }) { // roomId 是响应式的
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // serverUrl 是响应式的
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]); // ✅ 声明的所有依赖
  // ...
}
```

在上面的沙盒中尝试这个修复方法。验证一下是否消除了检查工具的错误，并且在需要时聊天会重新连接。

> ### 注意
>
> 在某些情况下，React **知道** 一个值永远不会改变，即使它在组件内部声明。例如，从 `useState` 返回的 `set` 函数和从 `useRef` 返回的 ref 对象是 **稳定的** ——它们保证在重新渲染时不会改变。稳定值不是响应式的，因此可以从列表中省略它们。包括它们是允许的：它们不会改变，所以无关紧要。

### 当你不想进行重新同步时该怎么办 

在上一个示例中，通过将 `roomId` 和 `serverUrl` 列为依赖项来修复了 lint 错误。

然而，可以通过向检查工具“证明”这些值不是响应式值，即它们 **不会** 因为重新渲染而改变。例如，如果 `serverUrl` 和 `roomId` 不依赖于渲染并且始终具有相同的值，可以将它们移到组件外部。现在它们不需要成为依赖项：

```js
function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ 声明的所有依赖
  // ...
}
```

也可以将它们 **移动到 Effect 内部**。它们不是在渲染过程中计算的，因此它们不是响应式的：

```js
function ChatRoom() {
  useEffect(() => {
    const serverUrl = 'https://localhost:1234'; // serverUrl 不是响应式的
    const roomId = 'general'; // roomId 不是响应式的
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ 声明的所有依赖
  // ...
}
```

**Effect 是一段响应式的代码块**。它们在读取的值发生变化时重新进行同步。与事件处理程序不同，事件处理程序只在每次交互时运行一次，而 Effect 则在需要进行同步时运行。

**不能“选择”依赖项**。依赖项必须包括 Effect 中读取的每个 [响应式值](https://zh-hans.react.dev/learn/lifecycle-of-reactive-effects#all-variables-declared-in-the-component-body-are-reactive)。代码检查工具会强制执行此规则。有时，这可能会导致出现无限循环的问题，或者 Effect 过于频繁地重新进行同步。不要通过禁用代码检查来解决这些问题！下面是一些解决方案：

- **检查 Effect 是否表示了独立的同步过程**。如果 Effect 没有进行任何同步操作，[可能是不必要的](https://zh-hans.react.dev/learn/you-might-not-need-an-effect)。如果它同时进行了几个独立的同步操作，[将其拆分为多个 Effect](https://zh-hans.react.dev/learn/lifecycle-of-reactive-effects#each-effect-represents-a-separate-synchronization-process)。
- **如果想读取 props 或 state 的最新值，但又不想对其做出反应并重新同步 Effect**，可以将 Effect 拆分为具有反应性的部分（保留在 Effect 中）和非反应性的部分（提取为名为 “Effect Event” 的内容）。[阅读关于将事件与 Effect 分离的内容](https://zh-hans.react.dev/learn/separating-events-from-effects)。
- **避免将对象和函数作为依赖项**。如果在渲染过程中创建对象和函数，然后在 Effect 中读取它们，它们将在每次渲染时都不同。这将导致 Effect 每次都重新同步。[阅读有关从 Effect 中删除不必要依赖项的更多内容](https://zh-hans.react.dev/learn/removing-effect-dependencies)。

> ### 陷阱
>
> 检查工具是你的朋友，但它们的能力是有限的。检查工具只知道依赖关系是否 **错误**。它并不知道每种情况下的 **最佳** 解决方法。如果静态代码分析工具建议添加某个依赖关系，但添加该依赖关系会导致循环，这并不意味着应该忽略静态代码分析工具。需要修改 Effect 内部（或外部）的代码，使得该值不是响应式的，也不 **需要** 成为依赖项。
>
> 如果有一个现有的代码库，可能会有一些像这样禁用了检查工具的 Effect：
>
> ```
> useEffect(() => {
>   // ...
>   // 🔴 避免这样禁用静态代码分析工具：
>   // eslint-ignore-next-line react-hooks/exhaustive-deps
> }, []);
> ```
>
> 在 [下一页](https://zh-hans.react.dev/learn/separating-events-from-effects) 和 [之后的页面](https://zh-hans.react.dev/learn/removing-effect-dependencies) 中，你将学习如何修复这段代码，而不违反规则。修复代码总是值得的！

## 摘要

- 组件可以挂载、更新和卸载。
- 每个 Effect 与周围组件有着独立的生命周期。
- 每个 Effect 描述了一个独立的同步过程，可以 **开始** 和 **停止**。
- 在编写和读取 Effect 时，要独立地考虑每个 Effect（如何开始和停止同步），而不是从组件的角度思考（如何挂载、更新或卸载）。
- 在组件主体内声明的值是“响应式”的。
- 响应式值应该重新进行同步 Effect，因为它们可以随着时间的推移而发生变化。
- 检查工具验证在 Effect 内部使用的所有响应式值都被指定为依赖项。
- 检查工具标记的所有错误都是合理的。总是有一种方法可以修复代码，同时不违反规则。