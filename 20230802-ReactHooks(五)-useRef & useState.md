# 20230802-ReactHooks-useRef & useState

# useRef

`useRef` 是一个 React Hook，它能让你引用一个不需要渲染的值。

```js
const ref = useRef(initialValue)
```

## 参考 

`useRef(initialValue)` 

在你组件的顶层调用 `useRef` 声明一个 [ref](https://zh-hans.react.dev/learn/referencing-values-with-refs)。

```js
import { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(0);
  const inputRef = useRef(null);
  // ...
```

#### 参数 

- `initialValue`：ref 对象的 `current` 属性的初始值。可以是任意类型的值。这个参数会首次渲染后被忽略。

#### 返回值 

`useRef` 返回一个只有一个属性的对象:

- `current`：最初，它被设置为你传递的 `initialValue`。之后你可以把它设置为其他值。如果你把 ref 对象作为一个 JSX 节点的 `ref` 属性传递给 React，React 将为它设置 `current` 属性。

在后续的渲染中，`useRef` 将返回同一个对象。

#### 注意事项 

- 你可以修改 `ref.current` 属性。与 state 不同，它是可变的。然而，如果它持有一个用于渲染的对象（例如，你的 state 的一部分），那么你就不应该修改这个对象。
- 当你改变 `ref.current` 属性时，React 不会重新渲染你的组件。React 不知道你何时改变它，因为 ref 是一个普通的 JavaScript 对象。
- 除了 [初始化](https://zh-hans.react.dev/reference/react/useRef#avoiding-recreating-the-ref-contents) 外不要在渲染期间写入 **或者读取** `ref.current`。这会使你的组件的行为不可预测。
- 在严格模式下，React 将会 **调用两次组件方法**，这是为了 [帮助你发现意外的问题](https://zh-hans.react.dev/reference/react/useRef#my-initializer-or-updater-function-runs-twice)。这只是开发模式下的行为，不影响生产模式。每个 ref 对象将会创建两次，但是其中一个版本将被丢弃。如果你的组件函数是纯的（应该如此），这不会影响其行为。

## 使用方法 

### 用 ref 引用一个值 

在你的组件的顶层调用 `useRef` 声明一个或多个 [refs](https://zh-hans.react.dev/learn/referencing-values-with-refs)。

```js
import { useRef } from 'react';

function Stopwatch() {
  const intervalRef = useRef(0);
  // ...
```

`useRef` 返回一个具有单个 `current` 属性 的 ref 对象，并初始化为你提供的 initial value

在后续的渲染中，`useRef` 将返回相同的对象。你可以改变它的 `current` 属性来存储信息，并在之后读取它。这会让你联想起 [state](https://zh-hans.react.dev/reference/react/useState)，但是有一个重要的区别。

**改变 ref 不会触发重新渲染。** 这意味着 ref 是存储一些不影响组件视图输出的信息的完美选择。例如，如果你需要存储一个 [intervalID](https://developer.mozilla.org/zh-CN/docs/Web/API/setInterval) 并在以后检索它，你可以把它放在一个 ref 中。如果要更新 ref 里面的值，你需要手动改变它的 `current` 属性：

```js
function handleStartClick() {
  const intervalId = setInterval(() => {
    // ...
  }, 1000);
  intervalRef.current = intervalId;
}
```

在之后，你可以从 ref 中读取 interval ID，这样你就可以 [清除定时器](https://developer.mozilla.org/zh-CN/docs/Web/API/clearInterval)：

```js
function handleStopClick() {
  const intervalId = intervalRef.current;
  clearInterval(intervalId);
}
```

通过使用 ref，你可以确保：

- 你可以在重新渲染之间 **存储信息**（不像是普通对象，每次渲染都会重置）。
- 改变它 **不会触发重新渲染**（不像是 state 变量，会触发重新渲染）。
- 对于你的组件的每个副本来说，**这些信息都是本地的**（不像是外面的变量，是共享的）。

改变 ref 不会触发重新渲染，所以 ref 不适合用于存储期望显示在屏幕上的信息。如有需要，使用 state 代替。阅读更多关于 [在 `useRef` 和 `useState` 之间选择](https://zh-hans.react.dev/learn/referencing-values-with-refs#differences-between-refs-and-state) 的信息。

### 使用useRef引用值的示例

#### 点击计数器

这个组件使用 ref 来记录按钮被点击的次数。注意，在这里使用 ref 而不是 state 是可以的，因为点击次数只在事件处理程序中被读取和写入。

<iframe src="https://codesandbox.io/embed/upbeat-blackburn-lle79h?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="upbeat-blackburn-lle79h"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

如果你在 JSX 中显示 `{ref.current}`，数字不会在点击时更新。这是因为修改 `ref.current` 不会触发重新渲染。用于渲染的信息应该使用 state。

#### 秒表

这个例子使用了 state 和 ref 的组合。`startTime` 和 `now` 都是 state 变量，因为它们是用来渲染的。但是我们还需要持有一个 [interval ID](https://developer.mozilla.org/zh-CN/docs/Web/API/setInterval)，这样我们就可以在按下按钮时停止定时器。因为 interval ID 不用于渲染，所以应该把它保存在一个 ref 中，并且手动更新它。

<iframe src="https://codesandbox.io/embed/exciting-hertz-9trm60?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="exciting-hertz-9trm60"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 陷阱

**不要在渲染期间写入 \*或者读取\* `ref.current`。**

React 期望你的组件的主体 [表现得像一个纯函数](https://zh-hans.react.dev/learn/keeping-components-pure)：

- 如果输入的（[props](https://zh-hans.react.dev/learn/passing-props-to-a-component)、[state](https://zh-hans.react.dev/learn/state-a-components-memory) 和 [context](https://zh-hans.react.dev/learn/passing-data-deeply-with-context)）都是一样的，那么就应该返回一样的 JSX。
- 以不同的顺序或用不同的参数调用它，不应该影响其他调用的结果。

在 **渲染期间** 读取或写入 ref 会破坏这些预期行为。

```jsx
function MyComponent() {
  // ...
  // 🚩 不要在渲染期间写入 ref
  myRef.current = 123;
  // ...
  // 🚩 不要在渲染期间读取 ref
  return <h1>{myOtherRef.current}</h1>;
}
```

你可以在 **事件处理程序或者 effects** 中读取和写入 ref。

```js
function MyComponent() {
  // ...
  useEffect(() => {
    // ✅ 你可以在 effects 中读取和写入 ref
    myRef.current = 123;
  });
  // ...
  function handleClick() {
    // ✅ 你可以在事件处理程序中读取和写入 ref
    doSomething(myOtherRef.current);
  }
  // ...
}
```

如果 **不得不** 在渲染期间读取 [或者写入](https://zh-hans.react.dev/reference/react/useState#storing-information-from-previous-renders)，[使用 state](https://zh-hans.react.dev/reference/react/useState) 代替。

当你打破这些规则时，你的组件可能仍然可以工作，但是我们为 React 添加的大多数新功能将依赖于这些预期行为。阅读 [保持你的组件纯粹](https://zh-hans.react.dev/learn/keeping-components-pure#where-you-_can_-cause-side-effects) 以了解更多信息。

### 避免重复创建 ref 的内容 

React 保存首次的 ref 初始值，并在后续的渲染中忽略它。

```js
function Video() {
  const playerRef = useRef(new VideoPlayer());
  // ...
```

虽然 `new VideoPlayer()` 的结果只会在首次渲染时使用，但是你依然在每次渲染时都在调用这个方法。如果是创建昂贵的对象，这可能是一种浪费。

为了解决这个问题，你可以像这样初始化 ref：

```js
function Video() {
  const playerRef = useRef(null);
  if (playerRef.current === null) {
    playerRef.current = new VideoPlayer();
  }
  // ...
```

通常情况下，在渲染过程中写入或读取 `ref.current` 是不允许的。然而，在这种情况下是可以的，因为结果总是一样的，而且条件只在初始化时执行，所以是完全可以预测的。

##### 深入探讨: 如何避免在初始化 `useRef` 之后进行 `null` 的类型检查 

如果你使用一个类型检查器，并且不想总是检查 `null`，你可以尝试用这样的模式来代替：

```js
function Video() {
  const playerRef = useRef(null);

  function getPlayer() {
    if (playerRef.current !== null) {
      return playerRef.current;
    }
    const player = new VideoPlayer();
    playerRef.current = player;
    return player;
  }

  // ...
```

在这里，`playerRef` 本身是可以为空的。然而，你应该能够使你的类型检查器确信，不存在 `getPlayer()` 返回 `null` 的情况。然后在你的事件处理程序中使用 `getPlayer()`。

## 疑难解答 

### 我无法获取自定义组件的 ref 

如果你尝试像这样传递 `ref` 到你自己的组件：

```jsx
const inputRef = useRef(null);

return <MyInput ref={inputRef} />;
```

你可能会在控制台中得到这样的错误：

> **Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?**

默认情况下，你自己的组件不会暴露它们内部 DOM 节点的 ref。

为了解决这个问题，首先，找到你想获得 ref 的组件：

```jsx
export default function MyInput({ value, onChange }) {
  return (
    <input
      value={value}
      onChange={onChange}
    />
  );
}
```

然后像这样将其包装在 [`forwardRef`](https://zh-hans.react.dev/reference/react/forwardRef) 里：

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(({ value, onChange }, ref) => {
  return (
    <input
      value={value}
      onChange={onChange}
      ref={ref}
    />
  );
});

export default MyInput;
```

最后，父级组件就可以得到它的 ref。

阅读 [访问另一个组件的 DOM 节点](https://zh-hans.react.dev/learn/manipulating-the-dom-with-refs#accessing-another-components-dom-nodes) 了解更多信息。

# useState

`useState` 是一个 React Hook，它允许你向组件添加一个 [状态变量](https://zh-hans.react.dev/learn/state-a-components-memory)。

```js
const [state, setState] = useState(initialState);
```

## 参考 

### `useState(initialState)` 

在组件的顶层调用 `useState` 来声明一个 [状态变量](https://zh-hans.react.dev/learn/state-a-components-memory)。

```js
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(28);
  const [name, setName] = useState('Taylor');
  const [todos, setTodos] = useState(() => createTodos());
  // ...
```

按照惯例使用 [数组解构](https://javascript.info/destructuring-assignment) 来命名状态变量，例如 `[something, setSomething]`。

#### 参数 

- `initialState`：你希望 state 初始化的值。它可以是任何类型的值，但对于函数有特殊的行为。在初始渲染后，此参数将被忽略。
  - 如果传递函数作为 `initialState`，则它将被视为 **初始化函数**。它应该是纯函数，不应该接受任何参数，并且应该返回一个任何类型的值。当初始化组件时，React 将调用你的初始化函数，并将其返回值存储为初始状态。[请参见下面的示例](https://zh-hans.react.dev/reference/react/useState#avoiding-recreating-the-initial-state)。

#### 返回 

`useState` 返回一个由两个值组成的数组：

1. 当前的 state。在首次渲染时，它将与你传递的 `initialState` 相匹配。
2. [`set` 函数](https://zh-hans.react.dev/reference/react/useState#setstate)，它可以让你将 state 更新为不同的值并触发重新渲染。

### `set` 函数，例如 `setSomething(nextState)` 

`useState` 返回的 `set` 函数允许你将 state 更新为不同的值并触发重新渲染。你可以直接传递新状态，也可以传递一个根据先前状态来计算新状态的函数：

```js
const [name, setName] = useState('Edward');

function handleClick() {
  setName('Taylor');
  setAge(a => a + 1);
  // ...
```

#### 参数 

- `nextState`：你想要 state 更新为的值。它可以是任何类型的值，但对于函数有特殊的行为。
  - 如果你将函数作为 `nextState` 传递，它将被视为 **更新函数**。它必须是纯函数，只接受待定的 state 作为其唯一参数，并应返回下一个状态。React 将把你的更新函数放入队列中并重新渲染组件。在下一次渲染期间，React 将通过把队列中所有更新函数应用于先前的状态来计算下一个状态。[请参见下面的示例](https://zh-hans.react.dev/reference/react/useState#updating-state-based-on-the-previous-state)。

#### 返回值 

`set` 函数没有返回值。

#### 注意事项 

- `set` 函数 **仅更新 \*下一次\* 渲染的状态变量**。如果在调用 `set` 函数后读取状态变量，则 [仍会得到在调用之前显示在屏幕上的旧值](https://zh-hans.react.dev/reference/react/useState#ive-updated-the-state-but-logging-gives-me-the-old-value)。
- 如果你提供的新值与当前 `state` 相同（由 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较确定），React 将 **跳过重新渲染该组件及其子组件**。这是一种优化。虽然在某些情况下 React 仍然需要在跳过子组件之前调用你的组件，但这不应影响你的代码。
- React 会 [批量处理状态更新](https://zh-hans.react.dev/learn/queueing-a-series-of-state-updates)。它会在所有 **事件处理函数运行** 并调用其 `set` 函数后更新屏幕。这可以防止在单个事件期间多次重新渲染。在某些罕见情况下，你需要强制 React 更早地更新屏幕，例如访问 DOM，你可以使用 [`flushSync`](https://zh-hans.react.dev/reference/react-dom/flushSync)。
- 在渲染期间，只允许在当前渲染组件内部调用 `set` 函数。React 将丢弃其输出并立即尝试使用新状态重新渲染。这种方式很少需要，但你可以使用它来存储 **先前渲染中的信息**。[请参见下面的示例](https://zh-hans.react.dev/reference/react/useState#storing-information-from-previous-renders)。
- 在严格模式中，React 将 **两次调用你的更新函数**，以帮助你找到 [意外的不纯性](https://zh-hans.react.dev/reference/react/useState#my-initializer-or-updater-function-runs-twice)。这只是开发时的行为，不影响生产。如果你的更新函数是纯函数（本该是这样），就不应影响该行为。其中一次调用的结果将被忽略。

## 用法 

### 为组件添加状态 

在组件的顶层调用 `useState` 来声明一个或多个 [状态变量](https://zh-hans.react.dev/learn/state-a-components-memory)。

```js
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(42);
  const [name, setName] = useState('Taylor');
  // ...
```

按照惯例使用 [数组解构](https://javascript.info/destructuring-assignment) 来命名状态变量，例如 `[something, setSomething]`。

`useState` 返回一个只包含两个项的数组：

1. 该状态变量 当前的 state，最初设置为你提供的 初始化 state。
2. `set` 函数，它允许你在响应交互时将 state 更改为任何其他值。

要更新屏幕上的内容，请使用新状态调用 `set` 函数：

```js
function handleClick() {
  setName('Robin');
}
```

React 会存储新状态，使用新值重新渲染组件，并更新 UI。

### 陷阱

> 调用 `set` 函数 [**不会** 改变已经执行的代码中当前的 state](https://zh-hans.react.dev/reference/react/useState#ive-updated-the-state-but-logging-gives-me-the-old-value)：
>
> ```js
> function handleClick() {
>   setName('Robin');
>   console.log(name); // Still "Taylor"!
> }
> ```
>
> 它只影响 **下一次** 渲染中 `useState` 返回的内容。

### 根据先前的 state 更新 state 

假设 `age` 为 `42`，这个处理函数三次调用 `setAge(age + 1)`：

```js
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
```

然而，点击一次后，`age` 将只会变为 `43` 而不是 `45`！这是因为调用 `set` 函数 [不会更新](https://zh-hans.react.dev/learn/state-as-a-snapshot) 已经运行代码中的 `age` 状态变量。因此，每个 `setAge(age + 1)` 调用变成了 `setAge(43)`。

为了解决这个问题，**你可以向 `setAge` 传递一个 \*更新函数\***，而不是下一个状态：

```js
function handleClick() {
  setAge(a => a + 1); // setAge(42 => 43)
  setAge(a => a + 1); // setAge(43 => 44)
  setAge(a => a + 1); // setAge(44 => 45)
}
```

这里，`a => a + 1` 是更新函数。它获取 **待定状态** 并从中计算 **下一个状态**。

React 将更新函数放入 [队列](https://zh-hans.react.dev/learn/queueing-a-series-of-state-updates) 中。然后，在下一次渲染期间，它将按照相同的顺序调用它们：

1. `a => a + 1` 将接收 `42` 作为待定状态，并返回 `43` 作为下一个状态。
2. `a => a + 1` 将接收 `43` 作为待定状态，并返回 `44` 作为下一个状态。
3. `a => a + 1` 将接收 `44` 作为待定状态，并返回 `45` 作为下一个状态。

现在没有其他排队的更新，因此 React 最终将存储 `45` 作为当前状态。

按照惯例，通常将待定状态参数命名为状态变量名称的第一个字母，如 `age` 为 `a`。然而，你也可以把它命名为 `prevAge` 或者其他你觉得更清楚的名称。

React 在开发环境中可能会 [两次调用你的更新函数](https://zh-hans.react.dev/reference/react/useState#my-initializer-or-updater-function-runs-twice) 来验证其是否为 [纯函数](https://zh-hans.react.dev/learn/keeping-components-pure)。

##### 深入探讨: 是否总是优先使用更新函数？ 

> 你可能会听到这样的建议，如果要设置的状态是根据先前的状态计算得出的，则应始终编写类似于 `setAge(a => a + 1)` 的代码。这样做没有害处，但也不总是必要的。
>
> 在大多数情况下，这两种方法没有区别。React 始终确保对于有意的用户操作，如单击，`age` 状态变量将在下一次单击之前被更新。这意味着单击事件处理函数在事件处理开始没有得到“过时” `age` 的风险。
>
> 但是，如果在同一事件中进行多个更新，则更新函数可能会有帮助。如果访问状态变量本身不方便（在优化重新渲染时可能会遇到这种情况），它们也很有用。
>
> 如果比起轻微的冗余你更喜欢语法的一致性，你正设置的状态又是根据先前的状态计算出来的，那么总是编写一个更新函数是合理的。如果它是从某个其他状态变量的先前状态计算出的，则你可能希望将它们结合成一个对象然后 [使用 reducer](https://zh-hans.react.dev/learn/extracting-state-logic-into-a-reducer)。

### 更新状态中的对象和数组 

你可以将对象和数组放入状态中。在 React 中，状态被认为是只读的，因此 **你应该替换它而不是改变现有对象**。例如，如果你在状态中保存了一个 `form` 对象，请不要改变它：

```js
// 🚩 不要像下面这样改变一个对象：
form.firstName = 'Taylor';
```

相反，可以通过创建一个新对象来替换整个对象：

```js
// ✅ 使用新对象替换 state
setForm({
  ...form,
  firstName: 'Taylor'
});
```

阅读有关 [更新状态中的对象](https://zh-hans.react.dev/learn/updating-objects-in-state) 和 [更新状态中的数组](https://zh-hans.react.dev/learn/updating-arrays-in-state) 来了解更多。

### 避免重复创建初始状态 

React 只在初次渲染时保存初始状态，后续渲染时将其忽略。

```js
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos());
  // ...
```

尽管 `createInitialTodos()` 的结果仅用于初始渲染，但你仍然在每次渲染时调用此函数。如果它创建大数组或执行昂贵的计算，这可能会浪费资源。

为了解决这个问题，你可以将它 **作为初始化函数传递给** `useState`：

```js
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
  // ...
```

请注意，你传递的是 `createInitialTodos` **函数本身**，而不是 `createInitialTodos()` 调用该函数的结果。如果将函数传递给 `useState`，React 仅在初始化期间调用它。

React 在开发模式下可能会调用你的 [初始化函数](https://zh-hans.react.dev/reference/react/useState#my-initializer-or-updater-function-runs-twice) 两次，以验证它们是否是 [纯函数](https://zh-hans.react.dev/learn/keeping-components-pure)。

#### 传递初始化函数和直接传递初始状态之间的区别

#### 传递初始化函数

<iframe src="https://codesandbox.io/embed/interesting-bush-6h3drq?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="interesting-bush-6h3drq"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

#### 直接传递初始状态 

<iframe src="https://codesandbox.io/embed/upbeat-almeida-6rhlzs?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="upbeat-almeida-6rhlzs"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 使用 key 重置状态 

在 [渲染列表](https://zh-hans.react.dev/learn/rendering-lists) 时，你经常会遇到 `key` 属性。然而，它还有另外一个用途。

你可以 **通过向组件传递不同的 `key` 来重置组件的状态**。在这个例子中，重置按钮改变 `version` 状态变量，我们将它作为一个 `key` 传递给 `Form` 组件。当 `key` 改变时，React 会从头开始重新创建 `Form` 组件（以及它的所有子组件），所以它的状态被重置了。

阅读 [保留和重置状态](https://zh-hans.react.dev/learn/preserving-and-resetting-state) 以了解更多。

<iframe src="https://codesandbox.io/embed/youthful-shaw-30dv33?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="youthful-shaw-30dv33"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 存储前一次渲染的信息 

通常情况下，你会在事件处理函数中更新状态。然而，在极少数情况下，你可能希望在响应渲染时调整状态——例如，当 props 改变时，你可能希望改变状态变量。

在大多数情况下，你不需要这样做：

- **如果你需要的值可以完全从当前 props 或其他 state 中计算出来，那么 [完全可以移除那些多余的状态](https://zh-hans.react.dev/learn/choosing-the-state-structure#avoid-redundant-state)**。如果你担心重新计算的频率过高，可以使用 [`useMemo` Hook](https://zh-hans.react.dev/reference/react/useMemo) 来帮助优化。
- 如果你想重置整个组件树的状态，[可以向组件传递一个不同的 `key`](https://zh-hans.react.dev/reference/react/useState#resetting-state-with-a-key)。
- 如果可以的话，在事件处理函数中更新所有相关状态。

在极为罕见的情况下，如果上述方法都不适用，你还可以使用一种方式，在组件渲染时调用 `set` 函数来基于已经渲染的值更新状态。

这里是一个例子。这个 `CountLabel` 组件显示传递给它的 `count` props：

```jsx
export default function CountLabel({ count }) {
  return <h1>{count}</h1>
}
```

假设你想显示计数器是否自上次更改以来 **增加或减少**。`count` props 无法告诉你这一点——你需要跟踪它的先前值。添加 `prevCount` 状态变量来跟踪它，再添加另一个状态变量 `trend` 来保存计数是否增加或减少。比较 `prevCount` 和 `count`，如果它们不相等，则更新 `prevCount` 和 `trend`。现在你既可以显示当前的 `count` props，也可以显示 **自上次渲染以来它如何改变**。

<iframe src="https://codesandbox.io/embed/zealous-mountain-s1kx0r?fontsize=14&hidenavigation=1&module=%2FCountLabel.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="zealous-mountain-s1kx0r"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

请注意，在渲染时调用 `set` 函数时，它必须位于条件语句中，例如 `prevCount !== count`，并且必须在该条件语句中调用 `setPrevCount(count)`。否则，你的组件将在循环中重新渲染，直到崩溃。此外，你只能像这样更新 **当前渲染** 组件的状态。在渲染过程中调用 **另一个** 组件的 `set` 函数是错误的。最后，你的 `set` 调用仍应 [不直接改变 state 来更新](https://zh-hans.react.dev/reference/react/useState#updating-objects-and-arrays-in-state) 状态——这并不意味着你可以违反其他 [纯函数](https://zh-hans.react.dev/learn/keeping-components-pure) 的规则。

这种方式可能很难理解，通常最好避免使用。但是，它比在 effect 中更新状态要好。当你在渲染期间调用 `set` 函数时，React 将在你的组件使用 `return` 语句退出后立即重新渲染该组件，并在渲染子组件前进行。这样，子组件就不需要进行两次渲染。你的组件函数的其余部分仍会执行（然后结果将被丢弃）。如果你的条件判断在所有 Hook 调用的下方，可以提前添加一个 `return;` 以便更早地重新开始渲染。