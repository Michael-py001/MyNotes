# 20230801-ReactHooks-useCallback & useContext

## useCallback 

`useCallback` 是一个允许你在多次渲染中缓存函数的 React Hook。

```js
const cachedFn = useCallback(fn, dependencies)
```

### 参数 

- `fn`：想要缓存的函数。此函数可以接受任何参数并且返回任何值。React 将会在初次渲染而非调用时返回该函数。当进行下一次渲染时，如果 `dependencies` 相比于上一次渲染时没有改变，那么 React 将会返回相同的函数。否则，React 将返回在最新一次渲染中传入的函数，并且将其缓存以便之后使用。React 不会调用此函数，而是返回此函数。你可以自己决定何时调用以及是否调用。
- `dependencies`：有关是否更新 `fn` 的所有响应式值的一个列表。响应式值包括 props、state，和所有在你组件内部直接声明的变量和函数。如果你的代码检查工具 [配置了 React](https://zh-hans.react.dev/learn/editor-setup#linting)，那么它将校验每一个正确指定为依赖的响应式值。依赖列表必须具有确切数量的项，并且必须像 `[dep1, dep2, dep3]` 这样编写。React 使用 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较每一个依赖和它的之前的值。

### 返回值 

在初次渲染时，`useCallback` 返回你已经传入的 `fn` 函数

在之后的渲染中, 如果依赖没有改变，`useCallback` 返回上一次渲染中缓存的 `fn` 函数；否则返回这一次渲染传入的 `fn`。

### 注意 

- `useCallback` 是一个 Hook，所以应该在 **组件的顶层** 或自定义 Hook 中调用。你不应在循环或者条件语句中调用它。如果你需要这样做，请新建一个组件，并将 state 移入其中。
- 除非有特定的理由，React **将不会丢弃已缓存的函数**。

### 用法 

#### 跳过组件的重新渲染 

当你优化渲染性能的时候，有时需要缓存传递给子组件的函数。让我们先关注一下如何实现，稍后去理解在哪些场景中它是有用的。

为了缓存组件中多次渲染的函数，你需要将其定义在 `useCallback` Hook 中：

```js
import { useCallback } from 'react';

function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
  // ...
```

你需要传递两个参数给 `useCallback`：

1. 在多次渲染中需要缓存的函数
2. 函数内部需要使用到的所有组件内部值的 依赖列表。

初次渲染时，在 `useCallback` 处接收的 返回函数 将会是已经传入的函数。

在之后的渲染中，React 将会使用 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 把 当前的依赖 和已传入之前的依赖进行比较。如果没有任何依赖改变，`useCallback` 将会返回与之前一样的函数。否则 `useCallback` 将返回 **此次** 渲染中传递的函数。

简而言之，`useCallback` 在多次渲染中缓存一个函数，直至这个函数的依赖发生改变。

**让我们通过一个示例看看它何时有用**。

假设你正在从 `ProductPage` 传递一个 `handleSubmit` 函数到 `ShippingForm` 组件中：

```jsx
function ProductPage({ productId, referrer, theme }) {
  // ...
  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
```

注意，切换 `theme` props 后会让应用停滞一小会，但如果将 `<ShippingForm />` 从 JSX 中移除，应用将反应迅速。这就提示尽力优化 `ShippingForm` 组件将会很有用。

**默认情况下，当一个组件重新渲染时， React 将递归渲染它的所有子组件**，因此每当因 `theme` 更改时而 `ProductPage` 组件重新渲染时，`ShippingForm` 组件也会重新渲染。这对于不需要大量计算去重新渲染的组件来说影响很小。但如果你发现某次重新渲染很慢，你可以将 `ShippingForm` 组件包裹在 [`memo`](https://zh-hans.react.dev/reference/react/memo) 中。如果 props 和上一次渲染时相同，那么 `ShippingForm` 组件将跳过重新渲染。

```js
import { memo } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  // ...
});
```

**当代码像上面一样改变后，如果 props 与上一次渲染时相同，`ShippingForm` 将跳过重新渲染**。这时缓存函数就变得很重要。假设定义了 `handleSubmit` 而没有定义 `useCallback`：

```jsx
function ProductPage({ productId, referrer, theme }) {
  // 每当 theme 改变时，都会生成一个不同的函数
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }
  
  return (
    <div className={theme}>
      {/* 这将导致 ShippingForm props 永远都不会是相同的，并且每次它都会重新渲染 */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

与字面量对象 `{}` 总是会创建新对象类似，**在 JavaScript 中，`function () {}` 或者 `() => {}` 总是会生成不同的函数**。正常情况下，这不会有问题，但是这意味着 `ShippingForm` props 将永远不会是相同的，并且 [`memo`](https://zh-hans.react.dev/reference/react/memo) 对性能的优化永远不会生效。而这就是 `useCallback` 起作用的地方：

```jsx
function ProductPage({ productId, referrer, theme }) {
  // 在多次渲染中缓存函数
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // 只要这些依赖没有改变

  return (
    <div className={theme}>
      {/* ShippingForm 就会收到同样的 props 并且跳过重新渲染 */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

**将 `handleSubmit` 传递给 `useCallback` 就可以确保它在多次重新渲染之间是相同的函数**，直到依赖发生改变。注意，除非出于某种特定原因，否则不必将一个函数包裹在 `useCallback` 中。在本例中，你将它传递到了包裹在 [`memo`](https://zh-hans.react.dev/reference/react/memo) 中的组件，这允许它跳过重新渲染。不过还有其他场景可能需要用到 `useCallback`，本章将对此进行进一步描述。

> ### 注意
>
> **`useCallback` 只应作用于性能优化**。如果代码在没有它的情况下无法运行，请找到根本问题并首先修复它，然后再使用 `useCallback`。

##### 深入探讨: `useCallback` 与 `useMemo` 有何关系？ 

[`useMemo`](https://zh-hans.react.dev/reference/react/useMemo) 经常与 `useCallback` 一同出现。当尝试优化子组件时，它们都很有用。他们会 [记住](https://en.wikipedia.org/wiki/Memoization)（或者说，缓存）正在传递的东西：

```jsx
import { useMemo, useCallback } from 'react';

function ProductPage({ productId, referrer }) {
  const product = useData('/product/' + productId);

  const requirements = useMemo(() => { //调用函数并缓存结果
    return computeRequirements(product);
  }, [product]);

  const handleSubmit = useCallback((orderDetails) => { // 缓存函数本身
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);

  return (
    <div className={theme}>
      <ShippingForm requirements={requirements} onSubmit={handleSubmit} />
    </div>
  );
}
```

区别在于你需要缓存 **什么**:

- **[`useMemo`](https://zh-hans.react.dev/reference/react/useMemo) 缓存函数调用的结果**。在这里，它缓存了调用 `computeRequirements(product)` 的结果。除非 `product` 发生改变，否则它将不会发生变化。这让你向下传递 `requirements` 时而无需不必要地重新渲染 `ShippingForm`。必要时，React 将会调用传入的函数重新计算结果。
- **`useCallback` 缓存函数本身**。不像 `useMemo`，它不会调用你传入的函数。相反，它缓存此函数。从而除非 `productId` 或 `referrer` 发生改变，`handleSubmit` 自己将不会发生改变。这让你向下传递 `handleSubmit` 函数而无需不必要地重新渲染 `ShippingForm`。直至用户提交表单，你的代码都将不会运行。

如果你已经熟悉了 [`useMemo`](https://zh-hans.react.dev/reference/react/useMemo)，你可能发现将 `useCallback` 视为以下内容会很有帮助：

```js
// 在 React 内部的简化实现
function useCallback(fn, dependencies) {
  return useMemo(() => fn, dependencies);
}
```

[阅读更多关于 `useMemo` 与 `useCallback` 之间区别的信息](https://zh-hans.react.dev/reference/react/useMemo#memoizing-a-function)。

##### 深入探讨: 是否应该在任何地方添加 `useCallback`？ 

如果你的应用程序与本网站类似，并且大多数交互都很粗糙（例如替换页面或整个部分），则通常不需要缓存。另一方面，如果你的应用更像是一个绘图编辑器，并且大多数交互都是精细的（如移动形状），那么你可能会发现缓存非常有用。

使用 `useCallback` 缓存函数仅在少数情况下有意义：

- 将其作为 props 传递给包装在 [`memo`] 中的组件。如果 props 未更改，则希望跳过重新渲染。缓存允许组件仅在依赖项更改时重新渲染。
- 传递的函数可能作为某些 Hook 的依赖。比如，另一个包裹在 `useCallback` 中的函数依赖于它，或者依赖于 [`useEffect`](https://zh-hans.react.dev/reference/react/useEffect) 中的函数。

在其他情况下，将函数包装在 `useCallback` 中没有任何意义。

### 从记忆化回调中更新 state 

有时，你可能在记忆化回调汇中基于之前的 state 来更新 state。

下面的 `handleAddTodo` 函数将 `todos` 指定为依赖项，因为它会从中计算下一个 todos：

```js
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos([...todos, newTodo]);
  }, [todos]);
  // ...
```

我们期望记忆化函数具有尽可能少的依赖，当你读取 state 只是为了计算下一个 state 时，你可以通过传递 [updater function](https://zh-hans.react.dev/reference/react/useState#updating-state-based-on-the-previous-state) 以移除该依赖：

```js
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos(todos => [...todos, newTodo]);
  }, []); // ✅ 不需要 todos 依赖项
  // ...
```

在这里，并不是将 `todos` 作为依赖项并在内部读取它，而是传递一个关于 **如何** 更新 state 的指示器 (`todos => [...todos, newTodo]`) 给 React。[阅读更多有关 updater function 的内容](https://zh-hans.react.dev/reference/react/useState#updating-state-based-on-the-previous-state)。

### 防止频繁触发 Effect 

有时，你想要在 [Effect](https://zh-hans.react.dev/learn/synchronizing-with-effects) 内部调用函数：

```js
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  function createOptions() {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    // ...
```

这会产生一个问题，[每一个响应值都必须声明为 Effect 的依赖](https://zh-hans.react.dev/learn/lifecycle-of-reactive-effects#react-verifies-that-you-specified-every-reactive-value-as-a-dependency)。但是如果将 `createOptions` 声明为依赖，它会导致 Effect 不断重新连接到聊天室：

```js
 useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // 🔴 问题：这个依赖在每一次渲染中都会发生改变
  // ...
```

为了解决这个问题，需要在 Effect 中将要调用的函数包裹在 `useCallback` 中：

```js
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const createOptions = useCallback(() => {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }, [roomId]); // ✅ 仅当 roomId 更改时更改

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // ✅ 仅当 createOptions 更改时更改
  // ...
```

这将确保如果 `roomId` 相同，`createOptions` 在多次渲染中会是同一个函数。**但是，最好消除对函数依赖项的需求**。将你的函数移入 Effect **内部**：

```js
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    function createOptions() { // ✅ 无需使用回调或函数依赖！
      return {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
    }

    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ 仅当 roomId 更改时更改
  // ...
```

现在你的代码变得更简单了并且不需要 `useCallback`。[阅读更多关于移除 Effect 依赖的信息](https://zh-hans.react.dev/learn/removing-effect-dependencies#move-dynamic-objects-and-functions-inside-your-effect)。

### 优化自定义 Hook

如果你正在编写一个 [自定义 Hook](https://zh-hans.react.dev/learn/reusing-logic-with-custom-hooks)，建议将它返回的任何函数包裹在 `useCallback` 中：

```js
function useRouter() {
  const { dispatch } = useContext(RouterStateContext);

  const navigate = useCallback((url) => {
    dispatch({ type: 'navigate', url });
  }, [dispatch]);

  const goBack = useCallback(() => {
    dispatch({ type: 'back' });
  }, [dispatch]);

  return {
    navigate,
    goBack,
  };
}
```

这确保了 Hook 的使用者在需要时能够优化自己的代码。

## useContext

`useContext` 是一个 React Hook，可以让你读取和订阅组件中的 [context](https://zh-hans.react.dev/learn/passing-data-deeply-with-context)。

```js
const value = useContext(SomeContext)
```

在组件的顶层调用 `useContext` 来读取和订阅 [context](https://zh-hans.react.dev/learn/passing-data-deeply-with-context)。

```js
import { useContext } from 'react';

function MyComponent() {
  const theme = useContext(ThemeContext);
  // ...
```

### 参数 

- `SomeContext`：先前用 [`createContext`](https://zh-hans.react.dev/reference/react/createContext) 创建的 context。context 本身不包含信息，它只代表你可以提供或从组件中读取的信息类型。

### 返回值 

`useContext` 为调用组件返回 context 的值。它被确定为传递给树中调用组件上方最近的 `SomeContext.Provider` 的 `value`。如果没有这样的 provider，那么返回值将会是为创建该 context 传递给 [`createContext`](https://zh-hans.react.dev/reference/react/createContext) 的 `defaultValue`。返回的值始终是最新的。如果 context 发生变化，React 会自动重新渲染读取 context 的组件。

### 注意事项 

- 组件中的 `useContext()` 调用不受 **同一** 组件返回的 provider 的影响。相应的 `<Context.Provider>` 需要位于调用 `useContext()` 的组件 **之上**。
- 从 provider 接收到不同的 `value` 开始，React 自动重新渲染使用了该特定 context 的所有子级。先前的值和新的值会使用 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 来做比较。使用 [`memo`](https://zh-hans.react.dev/reference/react/memo) 来跳过重新渲染并不妨碍子级接收到新的 context 值。
- 如果您的构建系统在输出中产生重复的模块（可能发生在符号链接中），这可能会破坏 context。通过 context 传递数据只有在用于传递 context 的 `SomeContext` 和用于读取数据的 `SomeContext` 是完全相同的对象时才有效，这是由 `===` 比较决定的。

### 用法 

#### 向组件树深层传递数据 

在组件的最顶级调用 `useContext` 来读取和订阅 [context](https://zh-hans.react.dev/learn/passing-data-deeply-with-context)。

```js
import { useContext } from 'react';

function Button() {
  const theme = useContext(ThemeContext);
  // ...
```

`useContext` 返回你向 context 传递的 context value。为了确定 context 值，React 搜索组件树，为这个特定的 context **向上查找最近的** context provider。

若要将 context 传递给 `Button`，请将其或其父组件之一包装到相应的 context provider：

```jsx
function MyPage() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  );
}

function Form() {
  // ... 在内部渲染 buttons ...
}
```

#### 陷阱

`useContext()` 总是在调用它的组件 **上面** 寻找最近的 provider。它向上搜索， **不考虑** 调用 `useContext()` 的组件中的 provider。

### 通过 context 更新传递的数据 

通常，你会希望 context 随着时间的推移而改变。要更新 context，请将其与 [state](https://zh-hans.react.dev/reference/react/useState) 结合。在父组件中声明一个状态变量，并将当前状态作为 context value 传递给 provider。

```jsx
function MyPage() {
  const [theme, setTheme] = useState('dark');
  return (
    <ThemeContext.Provider value={theme}>
      <Form />
      <Button onClick={() => {
        setTheme('light');
      }}>
       Switch to light theme
      </Button>
    </ThemeContext.Provider>
  );
}
```

现在 provider 中的任何一个 `Button` 都会接收到当前的 `theme` 值。如果调用 `setTheme` 来更新传递给 provider 的 `theme` 值，则所有 `Button` 组件都将使用新的值 `'light'` 来重新渲染。

### 指定回退默认值 

如果 React 没有在父树中找到该特定 context 的任何 provider，`useContext()` 返回的 context 值将等于你在 [创建 context](https://zh-hans.react.dev/reference/react/createContext) 时指定的 默认值：

```js
const ThemeContext = createContext(null);
```

默认值 **从不改变**。如果你想要更新 context，请按 [上述方式](https://zh-hans.react.dev/reference/react/useContext#updating-data-passed-via-context) 将其与状态一起使用。

通常，除了 `null`，还有一些更有意义的值可以用作默认值，例如：

```js
const ThemeContext = createContext('light');
```

这样，如果你不小心渲染了没有相应 provider 的某个组件，它也不会出错。这也有助于你的组件在测试环境中很好地运行，而无需在测试中设置许多 provider。

### 覆盖组件树一部分的 context 

通过在 provider 中使用不同的值包装树的某个部分，可以覆盖该部分的 context。

```jsx
<ThemeContext.Provider value="dark">
  ...
  <ThemeContext.Provider value="light">
    <Footer />
  </ThemeContext.Provider>
  ...
</ThemeContext.Provider>
```

你可以根据需要多次嵌套和覆盖 provider。

### 在传递对象和函数时优化重新渲染 

你可以通过 context 传递任何值，包括对象和函数。

```jsx
function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  function login(response) {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }

  return (
    <AuthContext.Provider value={{ currentUser, login }}>
      <Page />
    </AuthContext.Provider>
  );
}
```

此处，context value 是一个具有两个属性的 JavaScript 对象，其中一个是函数。每当 `MyApp` 出现重新渲染（例如，路由更新）时，这里将会是一个 **不同的** 对象指向 **不同的** 函数，因此 React 还必须重新渲染树中调用 `useContext(AuthContext)` 的所有组件。

在较小的应用程序中，这不是问题。但是，如果基础数据如 `currentUser` 没有更改，则不需要重新渲染它们。为了帮助 React 利用这一点，你可以使用 [`useCallback`](https://zh-hans.react.dev/reference/react/useCallback) 包装 `login` 函数，并将对象创建包装到 [`useMemo`](https://zh-hans.react.dev/reference/react/useMemo) 中。这是一个性能优化的例子：

```jsx
import { useCallback, useMemo } from 'react';

function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  const login = useCallback((response) => {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }, []);

  const contextValue = useMemo(() => ({
    currentUser,
    login
  }), [currentUser, login]);

  return (
    <AuthContext.Provider value={contextValue}>
      <Page />
    </AuthContext.Provider>
  );
}
```

根据以上改变，即使 `MyApp` 需要重新渲染，调用 `useContext(AuthContext)` 的组件也不需要重新渲染，除非 `currentUser` 发生了变化。

阅读更多关于 [`useMemo`](https://zh-hans.react.dev/reference/react/useMemo#skipping-re-rendering-of-components) 和 [`useCallback`](https://zh-hans.react.dev/reference/react/useCallback#skipping-re-rendering-of-components) 的内容。