# 20230802-ReactHooks-useMemo & useReducer

## useMemo

`useMemo` 是一个 React Hook，它在每次重新渲染的时候能够缓存计算的结果。

### 参考 

```js
const cachedValue = useMemo(calculateValue, dependencies)
```

在组件的顶层调用 `useMemo` 来缓存每次重新渲染都需要计算的结果。

```js
import { useMemo } from 'react';

function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );
  // ...
}
```

### 参数 

- `calculateValue`：要缓存计算值的函数。它应该是一个没有任何参数的纯函数，并且可以返回任意类型。React 将会在首次渲染时调用该函数；在之后的渲染中，如果 `dependencies` 没有发生变化，React 将直接返回相同值。否则，将会再次调用 `calculateValue` 并返回最新结果，然后缓存该结果以便下次重复使用。
- `dependencies`：所有在 `calculateValue` 函数中使用的响应式变量组成的数组。响应式变量包括 props、state 和所有你直接在组件中定义的变量和函数。如果你在代码检查工具中 [配置了 React](https://zh-hans.react.dev/learn/editor-setup#linting)，它将会确保每一个响应式数据都被正确地定义为依赖项。依赖项数组的长度必须是固定的并且必须写成 `[dep1, dep2, dep3]` 这种形式。React 使用 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 将每个依赖项与其之前的值进行比较。

### 返回值 

在初次渲染时，`useMemo` 返回不带参数调用 `calculateValue` 的结果。

在接下来的渲染中，如果依赖项没有发生改变，它将返回上次缓存的值；否则将再次调用 `calculateValue`，并返回最新结果。

> ### 注意
>
> 这种缓存返回值的方式也叫做 [记忆化（memoization）](https://en.wikipedia.org/wiki/Memoization)，这也是该 Hook 叫做 `useMemo` 的原因。

### 用法 

#### 跳过代价昂贵的重新计算 

在组件顶层调用 `useMemo` 以在重新渲染之间缓存计算结果：

```js
import { useMemo } from 'react';

function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

你需要给 `useMemo` 传递两样东西：

1. 一个没有任何参数的 **calculation 函数**，像这样 `() =>`，并且返回任何你想要的计算结果。
2. 一个由包含在你的组件中并在 calculation 中使用的所有值组成的 **依赖列表**。

在初次渲染时，你从 `useMemo` 得到的 **值** 将会是你的 **calculation** 函数执行的结果。

在随后的每一次渲染中，React 将会比较前后两次渲染中的 **所有依赖项** 是否相同。如果通过 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较所有依赖项都没有发生变化，那么 `useMemo` 将会返回之前已经计算过的那个值。否则，React 将会重新执行 calculation 函数并且返回一个新的值。

换言之，`useMemo` 在多次重新渲染中缓存了 calculation 函数计算的结果直到依赖项的值发生变化。

**让我们通过一个示例来看看这在什么情况下是有用的**。

默认情况下，React 会在每次重新渲染时重新运行整个组件。例如，如果 `TodoList` 更新了 state 或从父组件接收到新的 props，`filterTodos` 函数将会重新运行：

```js
function TodoList({ todos, tab, theme }) {
  const visibleTodos = filterTodos(todos, tab);
  // ...
}
```

如果计算速度很快，这将不会产生问题。但是，当正在过滤转换一个大型数组，或者进行一些昂贵的计算，而数据没有改变，那么可能希望跳过这些重复计算。如果 `todos` 与 `tab` 都与上次渲染时相同，那么像之前那样将计算函数包装在 `useMemo` 中，便可以重用已经计算过的 `visibleTodos`。

这种缓存行为叫做 [记忆化](https://en.wikipedia.org/wiki/Memoization)。

> ### 注意
>
> **你应该仅仅把 `useMemo` 作为性能优化的手段**。如果没有它，你的代码就不能正常工作，那么请先找到潜在的问题并修复它。然后再添加 `useMemo` 以提高性能。

##### 深入探讨: 如何衡量计算过程的开销是否昂贵？ 

一般来说，除非要创建或循环遍历数千个对象，否则开销可能并不大。如果你想获得更详细的信息，可以在控制台来测量花费这上面的时间：

```js
console.time('filter array');
const visibleTodos = filterTodos(todos, tab);
console.timeEnd('filter array');
```

然后执行你正在监测的交互（例如，在输入框中输入文字）。你将会在控制台看到如下的日志 `filter array: 0.15ms`。如果全部记录的时间加起来很长（`1ms` 或者更多），那么记忆此计算结果是有意义的。作为对比，你可以将计算过程包裹在 `useMemo` 中，以验证该交互的总日志时间是否减少了：

```js
console.time('filter array');
const visibleTodos = useMemo(() => {
  return filterTodos(todos, tab); // 如果 todos 和 tab 都没有变化，那么将会跳过渲染。
}, [todos, tab]);
console.timeEnd('filter array');
```

`useMemo` 不会让首次渲染更快，它只会帮助你跳过不必要的更新工作。

请记住，你的开发设备可能比用户的设备性能更强大，因此最好人为降低当前浏览器性能来测试。例如，Chrome 提供了 [CPU Throttling](https://developer.chrome.com/blog/new-in-devtools-61/#throttling) 选项来降低浏览器性能。

另外，请注意，在开发环境中测量性能无法为你提供最准确的结果（例如，当开启 [严格模式](https://zh-hans.react.dev/reference/react/StrictMode) 时，你会看到每个组件渲染两次而不是一次）。要获得最准确的时间，请构建用于生产的应用程序并在用户使用的设备上对其进行测试。

##### 深入探讨: 你应该在所有地方添加 useMemo 吗？ 

如果你的应用程序类似于此站点，并且大多数交互都很粗糙（例如替换页面或整个章节），则通常不需要使用记忆化。反之，如果你的应用程序更像是绘图编辑器，并且大多数交互都是颗粒状的（如移动形状），那么你可能会发现记忆化非常有用。

使用 `useMemo` 进行优化仅在少数情况下有价值：

- 你在 `useMemo` 中进行的计算明显很慢，而且它的依赖关系很少改变。
- 将计算结果作为 props 传递给包裹在 [`memo`](https://zh-hans.react.dev/reference/react/memo) 中的组件。当计算结果没有改变时，你会想跳过重新渲染。记忆化让组件仅在依赖项不同时才重新渲染。
- 你传递的值稍后用作某些 Hook 的依赖项。例如，也许另一个 `useMemo` 计算值依赖它，或者 [`useEffect`](https://zh-hans.react.dev/reference/react/useEffect) 依赖这个值。

在其他情况下，将计算过程包装在 `useMemo` 中没有任何好处。不过这样做也没有重大危害，所以一些团队选择不考虑具体情况，尽可能多地使用 `useMemo`。不过这种做法会降低代码可读性。此外，并不是所有 `useMemo` 的使用都是有效的：一个“永远是新的”的单一值就足以破坏整个组件的记忆化效果。

**在实践中，你可以通过遵循一些原则来避免 `useMemo` 的滥用**：

1. 当一个组件在视觉上包裹其他组件时，让它 [将 JSX 作为子组件传递](https://zh-hans.react.dev/learn/passing-props-to-a-component#passing-jsx-as-children)。这样，当包装器组件更新自己的 state 时，React 知道它的子组件不需要重新渲染。
2. 首选本地 state，非必要不进行 [状态提升](https://zh-hans.react.dev/learn/sharing-state-between-components)。例如，不要保持像表单、组件是否悬停在组件树顶部这样的瞬时状态。
3. 保持你的 [渲染逻辑纯粹](https://zh-hans.react.dev/learn/keeping-components-pure)。如果重新渲染组件会导致一些问题或产生一些明显的视觉错误，那么它就是组件中的错误！修复错误而不是使用记忆化。
4. 避免 [不必要地更新 state 的 Effect](https://zh-hans.react.dev/learn/you-might-not-need-an-effect)。React 应用程序中的大多数性能问题都是由 Effect 创造的更新链引起的，这些更新链导致组件反复重新渲染。
5. 尽力 [从 Effect 中移除不必要的依赖项](https://zh-hans.react.dev/learn/removing-effect-dependencies)。例如, 相比于记忆化，在 Effect 内部或组件外部移动某些对象或函数通常更简单。

#### 跳过组件的重新渲染

在某些情况下，`useMemo` 还可以帮助你优化重新渲染子组件的性能。为了说明这一点，假设这个 `TodoList` 组件将 `visibleTodos` 作为 props 传递给子 `List` 组件：

```jsx
export default function TodoList({ todos, tab, theme }) {
  // ...
  return (
    <div className={theme}>
      <List items={visibleTodos} />
    </div>
  );
}
```

你已经注意到切换 `theme` 属性会使应用程序冻结片刻，但是如果你从 JSX 中删除 `<List />`，感觉会很快。这说明尝试优化 `List` 组件是值得的。

**默认情况下，当一个组件重新渲染时，React 会递归地重新渲染它的所有子组件**。这就是为什么当 `TodoList` 使用不同的 `theme` 重新渲染时，`List` 组件 **也会** 重新渲染。这对于不需要太多计算来重新渲染的组件来说很好。但是如果你已经确认重新渲染很慢，你可以通过将它包装在 [`memo`](https://zh-hans.react.dev/reference/react/memo) 中，这样当它的 props 跟上一次渲染相同的时候它就会跳过本次渲染：

```js
import { memo } from 'react';

const List = memo(function List({ items }) {
  // ...
});
```

**通过此更改，如果 `List` 的所有 props 都与上次渲染时相同，则 `List` 将跳过重新渲染**。这就是缓存计算变得重要的地方！想象一下，你在没有 `useMemo` 的情况下计算了 `visibleTodos`：

```jsx
export default function TodoList({ todos, tab, theme }) {
  // 每当主题发生变化时，这将是一个不同的数组……
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      {/* ... 所以List的props永远不会一样，每次都会重新渲染 */}
      <List items={visibleTodos} />
    </div>
  );
}
```

**在上面的示例中，`filterTodos` 函数总是创建一个不同数组**，类似于 `{}` 总是创建一个新对象的方式。通常，这不是问题，但这意味着 `List` 属性永远不会相同，并且你的 [`memo`](https://zh-hans.react.dev/reference/react/memo) 优化将不起作用。这就是 `useMemo` 派上用场的地方：

```jsx
export default function TodoList({ todos, tab, theme }) {
  // 告诉 React 在重新渲染之间缓存你的计算结果...
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab] // ...所以只要这些依赖项不变...
  );
  return (
    <div className={theme}>
      {/* ... List 也就会接受到相同的 props 并且会跳过重新渲染 */}
      <List items={visibleTodos} />
    </div>
  );
}
```

**通过将 `visibleTodos` 的计算函数包裹在 `useMemo` 中，你可以确保它在重新渲染之间具有相同值**，直到依赖项发生变化。你 **不必** 将计算函数包裹在 `useMemo` 中，除非你出于某些特定原因这样做。在此示例中，这样做的原因是你将它传递给包裹在 [`memo`](https://zh-hans.react.dev/reference/react/memo) 中的组件，这使得它可以跳过重新渲染。添加 `useMemo` 的其他一些原因将在本页进一步描述。

##### 深入探讨: 记忆单个 JSX 节点 

你可以将 `<List />` JSX 节点本身包裹在 `useMemo` 中，而不是将 `List` 包裹在 [`memo`](https://zh-hans.react.dev/reference/react/memo) 中：

```jsx
export default function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  const children = useMemo(() => <List items={visibleTodos} />, [visibleTodos]);
  return (
    <div className={theme}>
      {children}
    </div>
  );
}
```

他们的行为表现是一致的。如果 `visibleTodos` 没有改变，`List` 将不会重新渲染。

像 `<List items={visibleTodos} />` 这样的 JSX 节点是一个类似 `{ type: List, props: { items: visibleTodos } }` 的对象。创建这个对象的开销很低，但是 React 不知道它的内容是否和上次一样。这就是为什么默认情况下，React 会重新渲染 `List` 组件。

但是，如果 React 发现其与之前渲染的 JSX 是完全相同的，它不会尝试重新渲染你的组件。这是因为 JSX 节点是 [不可变的（immutable）](https://en.wikipedia.org/wiki/Immutable_object)。JSX 节点对象不可能随时间改变，因此 React 知道跳过重新渲染是安全的。然而，为了使其工作，节点必须 **实际上是同一个对象**，而不仅仅是在代码中看起来相同。这就是 `useMemo` 在此示例中所做的。

手动将 JSX 节点包裹到 `useMemo` 中并不方便，比如你不能在条件语句中这样做。这就是为什么通常会选择使用 [`memo`](https://zh-hans.react.dev/reference/react/memo) 包装组件而不是使用 `useMemo` 包装 JSX 节点。

> ### memo
>
> ```js
> const MemoizedComponent = memo(SomeComponent, arePropsEqual?)
> ```

#### 记忆另一个 Hook 的依赖 

假设你有一个计算函数依赖于直接在组件主体中创建的对象：

```js
function Dropdown({ allItems, text }) {
  const searchOptions = { matchMode: 'whole-word', text };

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // 🚩 提醒：依赖于在组件主体中创建的对象
  // ...
```

依赖这样的对象会破坏记忆化。当组件重新渲染时，组件主体内的所有代码都会再次运行。**创建 `searchOptions` 对象的代码行也将在每次重新渲染时运行**。因为 `searchOptions` 是你的 `useMemo` 调用的依赖项，而且每次都不一样，React 知道依赖项是不同的，并且每次都重新计算 `searchItems`。

要解决此问题，你可以在将其作为依赖项传递之前记忆 `searchOptions` 对象 **本身**：

```jsx
function Dropdown({ allItems, text }) {
  const searchOptions = useMemo(() => {
    return { matchMode: 'whole-word', text };
  }, [text]); // ✅ 只有当 text 改变时才会发生改变

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // ✅ 只有当 allItems 或 serachOptions 改变时才会发生改变
  // ...
```

在上面的例子中，如果 `text` 没有改变，`searchOptions` 对象也不会改变。然而，更好的解决方法是将 `searchOptions` 对象声明移到 `useMemo` 计算函数的 **内部**：

```js
function Dropdown({ allItems, text }) {
  const visibleItems = useMemo(() => {
    const searchOptions = { matchMode: 'whole-word', text };
    return searchItems(allItems, searchOptions);
  }, [allItems, text]); // ✅ 只有当 allItems 或者 text 改变的时候才会重新计算
  // ...
```

现在你的计算直接取决于 `text`（这是一个字符串，不会“意外地”变得不同）。

#### 记忆一个函数 

假设 `Form` 组件被包裹在 [`memo`](https://zh-hans.react.dev/reference/react/memo) 中，你想将一个函数作为 props 传递给它：

```jsx
export default function ProductPage({ productId, referrer }) {
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails
    });
  }

  return <Form onSubmit={handleSubmit} />;
}
```

正如 `{}` 每次都会创建不同的对象一样，像 `function() {}` 这样的函数声明和像 `() => {}` 这样的表达式在每次重新渲染时都会产生一个 **不同** 的函数。就其本身而言，创建一个新函数不是问题。这不是可以避免的事情！但是，如果 `Form` 组件被记忆了，大概你想在没有 props 改变时跳过它的重新渲染。**总是** 不同的 props 会破坏你的记忆化。

要使用 `useMemo` 记忆函数，你的计算函数必须返回另一个函数：

```jsx
export default function Page({ productId, referrer }) {
  const handleSubmit = useMemo(() => {
    return (orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails
      });
    };
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

这看起来很笨拙！**记忆函数很常见，React 有一个专门用于此的内置 Hook。将你的函数包装到 [`useCallback`](https://zh-hans.react.dev/reference/react/useCallback) 而不是 `useMemo`** 中，以避免编写额外的嵌套函数：

```jsx
export default function Page({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails
    });
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

上面两个例子是完全等价的。`useCallback` 的唯一好处是它可以让你避免在内部编写额外的嵌套函数。它没有做任何其他事情。[阅读更多关于 `useCallback` 的内容](https://zh-hans.react.dev/reference/react/useCallback)。

### 总结

> - 缓存函数 -> `useCallback`
> - 缓存jsx组件 -> `memo`
> - 缓存数据 -> `useMemo`

## useReducer

`useReducer` 是一个 React Hook，它允许你向组件里面添加一个 [reducer](https://zh-hans.react.dev/learn/extracting-state-logic-into-a-reducer)。

### 参考 

`useReducer(reducer, initialArg, init?)` 

在组件的顶层作用域调用 `useReducer` 以创建一个用于**管理状态**的 [reducer](https://zh-hans.react.dev/learn/extracting-state-logic-into-a-reducer)。

### 参数 

- `reducer`：用于更新 state 的**纯函数**。参数为 **state 和 action**，**返回值**是**更新后的 state**。state 与 action 可以是任意合法值。
- `initialArg`：用于初始化 state 的任意值。初始值的计算逻辑取决于接下来的 `init` 参数。
- **可选参数** `init`：用于计算初始值的函数。如果存在，使用 `init(initialArg)` 的执行结果作为初始值，否则使用 `initialArg`。

### 返回值 

`useReducer` 返回一个由两个值组成的数组：

1. 当前的 state。初次渲染时，它是 `init(initialArg)` 或 `initialArg` （如果没有 `init` 函数）。
2. [`dispatch` 函数](https://zh-hans.react.dev/reference/react/useReducer#dispatch)。用于更新 state 并触发组件的重新渲染。

### `dispatch` 函数 

`useReducer` 返回的 `dispatch` 函数允许你更新 state 并触发组件的重新渲染。它需要传入一个 action 作为参数：

```js
const [state, dispatch] = useReducer(reducer, { age: 42 });

function handleClick() {
  dispatch({ type: 'incremented_age' });
  // ...
```

React 会调用 `reducer` 函数以更新 state，`reducer` 函数的参数为**当前的 state 与传递的 action**。

#### 参数 

- `action`：用户执行的操作。可以是任意类型的值。通常来说 action 是一个对象，其中 `type` 属性标识类型，其它属性携带额外信息。

#### 返回值 

`dispatch` 函数没有返回值。

#### 注意 

- `dispatch` 函数 **是为下一次渲染而更新 state**。因此在调用 `dispatch` 函数后读取 state [并不会拿到更新后的值](https://zh-hans.react.dev/reference/react/useReducer#ive-dispatched-an-action-but-logging-gives-me-the-old-state-value)，也就是说只能获取到调用前的值。
- 如果你提供的新值与当前的 `state` 相同（使用 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较），React 会 **跳过组件和子组件的重新渲染**，这是一种优化手段。虽然在跳过重新渲染前 React 可能会调用你的组件，但是这不应该影响你的代码。
- React [会批量更新 state](https://zh-hans.react.dev/learn/queueing-a-series-of-state-updates)。state 会在 **所有事件函数执行完毕** 并且已经调用过它的 `set` 函数后进行更新，这可以防止在一个事件中多次进行重新渲染。如果在访问 DOM 等极少数情况下需要强制 React 提前更新，可以使用 [`flushSync`](https://zh-hans.react.dev/reference/react-dom/flushSync)。

### 用法 

#### 向组件添加 reducer 

在组件的顶层作用域调用 `useReducer` 来创建一个用于管理状态（state）的 [reducer](https://zh-hans.react.dev/learn/extracting-state-logic-into-a-reducer)。

```js
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });
  // ...
```

`useReducer` 返回一个由两个值组成的数组：

1. **当前的 state**，首次渲染时为你提供的 **初始值**。
2. **`dispatch` 函数**，让你可以根据交互修改 state。

为了更新屏幕上的内容，使用一个表示用户操作的 action 来调用 `dispatch` 函数：

```js
function handleClick() {
  dispatch({ type: 'incremented_age' });
}
```

React 会把当前的 state 和这个 action 一起作为参数传给 **reducer 函数**，然后 reducer 计算并返回新的 state，最后 React 保存新的 state，并使用它渲染组件和更新 UI。

<iframe src="https://codesandbox.io/embed/hungry-firefly-mye6wj?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="hungry-firefly-mye6wj"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

`useReducer` 和 [`useState`](https://zh-hans.react.dev/reference/react/useState) 非常相似，但是它可以让你把状态更新逻辑从事件处理函数中移动到组件外部。详情可以参阅 [对比 `useState` 和 `useReducer`](https://zh-hans.react.dev/learn/extracting-state-logic-into-a-reducer#comparing-usestate-and-usereducer)。

#### 实现 reducer 函数 

reducer 函数的定义如下：

```js
function reducer(state, action) {
  // ...
}
```

你需要往函数体里面添加计算并返回新的 state 的逻辑。一般会使用 [`switch` 语句](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/switch) 来完成。在 `switch` 语句中通过匹配 `case` 条件来计算并返回相应的 state。

```js
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        name: state.name,
        age: state.age + 1
      };
    }
    case 'changed_name': {
      return {
        name: action.nextName,
        age: state.age
      };
    }
  }
  throw Error('Unknown action: ' + action.type);
}
```

action 可以是任意类型，不过通常**至少**是一个存在 **`type`** 属性的对象。也就是说它需要携带计算新的 state 值所必须的数据。

```js
function Form() {
  const [state, dispatch] = useReducer(reducer, { name: 'Taylor', age: 42 });
  
  function handleButtonClick() {
    dispatch({ type: 'incremented_age' });
  }

  function handleInputChange(e) {
    dispatch({
      type: 'changed_name',
      nextName: e.target.value
    });
  }
  // ...
```

action 的 type 依赖于组件的实际情况。[即使它会导致数据的多次更新，每个 action 都只描述一次交互](https://zh-hans.react.dev/learn/extracting-state-logic-into-a-reducer#writing-reducers-well)。state 的类型也是任意的，不过一般会使用对象或数组。

阅读 [迁移状态逻辑至 Reducer 中](https://zh-hans.react.dev/learn/extracting-state-logic-into-a-reducer) 来了解更多内容。

##### 陷阱

> state 是只读的。即使是对象或数组也不要尝试修改它：
>
> ```js
> function reducer(state, action) {
>   switch (action.type) {
>     case 'incremented_age': {
>       // 🚩 不要像下面这样修改一个对象类型的 state：
>       state.age = state.age + 1;
>       return state;
>     }
> ```
>
> 正确的做法是返回新的对象：
>
> ```js
> function reducer(state, action) {
>   switch (action.type) {
>     case 'incremented_age': {
>       // ✅ 正确的做法是返回新的对象
>       return {
>         ...state,
>         age: state.age + 1
>       };
>     }
> ```
>
> 阅读 [更新对象类型的 state](https://zh-hans.react.dev/learn/updating-objects-in-state) 和 [更新数组类型的 state](https://zh-hans.react.dev/learn/updating-arrays-in-state) 来了解更多内容。

##### useReducer 的基础示例

###### 表单（对象类型）

在这个示例中，state 是一个有 `name` 和 `age` 属性的对象。

<iframe src="https://codesandbox.io/embed/recursing-johnson-x51lij?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="recursing-johnson-x51lij"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

######  代办事项（数组类型）

在这个示例中，reducer 管理一个名为 tasks 的数组。数组 [不能使用修改方法](https://zh-hans.react.dev/learn/updating-arrays-in-state) 来更新。

<iframe src="https://codesandbox.io/embed/crazy-grass-v4k6mq?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="crazy-grass-v4k6mq"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

###### 使用 Immer 编写简洁的更新逻辑 

如果使用复制方法更新数组和对象让你不厌其烦，那么可以使用 [Immer](https://github.com/immerjs/use-immer#useimmerreducer) 这样的库来减少一些重复的样板代码。Immer 让你可以专注于逻辑，因为它在内部均使用复制方法来完成更新：

<iframe src="https://codesandbox.io/embed/hidden-wildflower-hshcri?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="hidden-wildflower-hshcri"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

#### 避免重新创建初始值 

React 会保存 state 的初始值并在下一次渲染时忽略它。

```js
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, createInitialState(username));
  // ...
```

虽然 `createInitialState(username)` 的返回值只用于初次渲染，但是在每一次渲染的时候都会被调用。如果它创建了比较大的数组或者执行了昂贵的计算就会浪费性能。

你可以通过给  `useReducer` 的第三个参数传入 **初始化函数** 来解决这个问题：

```js
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, username, createInitialState);
  // ...
```

需要注意的是你传入的参数是 `createInitialState` 这个 **函数自身**，而不是执行 `createInitialState()` 后的返回值。这样传参就可以保证初始化函数不会再次运行。

在上面这个例子中，`createInitialState` 有一个 `username` 参数。如果初始化函数不需要参数就可以计算出初始值，可以把 `useReducer` 的第二个参数改为 `null`。

##### 使用初始化函数和直接传入初始值的区别

###### 1、使用初始化函数 

这个示例使用了一个初始化函数，所以 `createInitialState` 函数只会在初次渲染的时候进行调用。即使往输入框中输入内容导致组件重新渲染，初始化函数也不会被再次调用。

<iframe src="https://codesandbox.io/embed/brave-chatelet-7scfy5?fontsize=14&hidenavigation=1&module=%2FTodoList.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="brave-chatelet-7scfy5"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

###### 2、直接传入初始值 

这个示例 **没有使用** 初始化函数，所以当你往输入框输入内容导致组件重新渲染的时候，`createInitialState` 函数就会执行。虽然在渲染结果上看没有什么区别，但是多余的逻辑会导致性能变差。

<iframe src="https://codesandbox.io/embed/lucid-ardinghelli-0moh1s?fontsize=14&hidenavigation=1&module=%2FTodoList.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="lucid-ardinghelli-0moh1s"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>