# 20230605-React学习-Ref与forwardRef操作DOM

> [使用 ref 操作 DOM – React](https://zh-hans.react.dev/learn/manipulating-the-dom-with-refs#getting-a-ref-to-the-node)

# 使用 ref 操作 DOM

​		你可能需要访问由 React 管理的 DOM 元素 —— 例如，让一个节点获得焦点、滚动到它或测量它的尺寸和位置。在 React 中没有内置的方法来做这些事情，所以你需要一个指向 DOM 节点的 *ref* 来实现。

## 获取指向节点的 ref 

- 要访问由 React 管理的 DOM 节点，首先，引入 `useRef` Hook：

  ```ts
  import { useRef } from 'react';
  ```

- 然后，在你的组件中使用它声明一个 ref：

  ```ts
  const myRef = useRef(null);
  ```

- 最后，将ref作为ref属性传递给要获取其DOM节点的JSX标记

  ```html
  <div ref={myRef}>
  ```

`useRef` Hook 返回一个对象，该对象有一个名为 `current` 的属性。最初，`myRef.current` 是 `null`。当 React 为这个 `<div>` 创建一个 DOM 节点时，React 会把对该节点的引用放入 `myRef.current`。然后，你可以从 [事件处理器](https://zh-hans.react.dev/learn/responding-to-events) 访问此 DOM 节点，并使用在其上定义的内置[浏览器 API](https://developer.mozilla.org/docs/Web/API/Element)。

```js
// 你可以使用任意浏览器 API，例如：
myRef.current.scrollIntoView();
```

## 使用forwardRef 访问另一个组件的 DOM 节点

当你将 ref 放在像 `<input />` 这样输出浏览器元素的内置组件上时，React 会将该 ref 的 `current` 属性设置为相应的 DOM 节点（例如浏览器中实际的 `<input />` ）。

但是，如果你尝试将 ref 放在 **你自己的** 组件上，例如 `<MyInput />`，默认情况下你会得到 `null`。

为了帮助您注意到这个问题，React 还会向控制台打印一条错误消息：

> **Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?**

发生这种情况是因为默认情况下，React 不允许组件访问其他组件的 DOM 节点。甚至自己的子组件也不行！这是故意的。Refs 是一个应急方案，应该谨慎使用。手动操作 *另一个* 组件的 DOM 节点会使你的代码更加脆弱。

相反，*想要* 暴露其 DOM 节点的组件必须**选择**该行为。一个组件可以指定将它的 ref “转发”给一个子组件。下面是 `MyInput` 如何使用 `forwardRef` API：

```js
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
```

它是这样工作的:

1. `<MyInput ref={inputRef} />` 告诉 React 将对应的 DOM 节点放入 `inputRef.current` 中。但是，这取决于 `MyInput` 组件是否允许这种行为， 默认情况下是不允许的。
2. `MyInput` 组件是使用 `forwardRef` 声明的。 **这让从上面接收的 `inputRef` 作为第二个参数 `ref` 传入组件**，第一个参数是 `props` 。
3. `MyInput` 组件将自己接收到的 `ref` 传递给它内部的 `<input>`。

现在，单击按钮聚焦输入框起作用了：

```jsx
import { forwardRef, useRef } from 'react';

const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        聚焦输入框
      </button>
    </>
  );
}

```

## useImperativeHandle  限制暴露的方法

在上面的例子中，`MyInput` 暴露了原始的 DOM 元素 input。这让父组件可以对其调用`focus()`。然而，这也让父组件能够做其他事情 —— 例如，改变其 CSS 样式。在一些不常见的情况下，你可能希望限制暴露的功能。你可以用 `useImperativeHandle` 做到这一点：

```jsx
import {
  forwardRef, 
  useRef, 
  useImperativeHandle
} from 'react';

const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // 只暴露 focus，没有别的
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        聚焦输入框
      </button>
    </>
  );
}

```

这里，`MyInput` 中的 `realInputRef` 保存了实际的 input DOM 节点。 但是，`useImperativeHandle` 指示 React 将你自己指定的对象作为父组件的 ref 值。 所以 `Form` 组件内的 `inputRef.current` 将只有 `focus` 方法。在这种情况下，ref “句柄”不是 DOM 节点，而是你在 `useImperativeHandle` 调用中创建的自定义对象。

## React 何时添加 refs 

在 React 中，每次更新都分为 [两个阶段](https://zh-hans.react.dev/learn/render-and-commit#step-3-react-commits-changes-to-the-dom)：

- 在 **渲染** 阶段， React 调用你的组件来确定屏幕上应该显示什么。
- 在 **提交** 阶段， React 把变更应用于 DOM。

通常，你 [不希望](https://zh-hans.react.dev/learn/referencing-values-with-refs#best-practices-for-refs) 在渲染期间访问 refs。这也适用于保存 DOM 节点的 refs。在第一次渲染期间，DOM 节点尚未创建，因此 `ref.current` 将为 `null`。在渲染更新的过程中，DOM 节点还没有更新。所以读取它们还为时过早。

React 在提交阶段设置 `ref.current`。在更新 DOM 之前，React 将受影响的 `ref.current` 值设置为 `null`。**更新 DOM 后**，React 立即将它们设置到相应的 DOM 节点。

**通常，你将从事件处理器访问 refs。** 如果你想使用 ref 执行某些操作，但没有特定的事件可以执行此操作，你可能需要一个 effect。我们将在下一页讨论 effect。

## 用 flushSync 同步更新 state

思考这样的代码，它添加一个新的待办事项，并将屏幕向下滚动到列表的最后一个子项。请注意，出于某种原因，它总是滚动到最后一个添加*之前* 的待办事项：

```jsx
import { useState, useRef } from 'react';

export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState('');
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    setText('');
    setTodos([ ...todos, newTodo]);
    listRef.current.lastChild.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest'
    });
  }

  return (
    <>
      <button onClick={handleAdd}>
        添加
      </button>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <ul ref={listRef}>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}

let nextId = 0;
let initialTodos = [];
for (let i = 0; i < 20; i++) {
  initialTodos.push({
    id: nextId++,
    text: '待办 #' + (i + 1)
  });
}

```

问题出在这两行：

```js
setTodos([ ...todos, newTodo]);
listRef.current.lastChild.scrollIntoView();
```

在 React 中，[state 更新是排队进行的](https://zh-hans.react.dev/learn/queueing-a-series-of-state-updates)。通常，这就是你想要的。但是，在这个示例中会导致问题，因为 `setTodos` 不会立即更新 DOM。因此，当你将列表滚动到最后一个元素时，尚未添加待办事项。这就是为什么滚动总是“落后”一项的原因。

要解决此问题，你可以强制 React 同步更新（“刷新”）DOM。 为此，从 `react-dom` 导入 `flushSync` 并**将 state 更新包裹** 到 `flushSync` 调用中：

```js
flushSync(() => {
  setTodos([ ...todos, newTodo]);
});
listRef.current.lastChild.scrollIntoView();
```

这将指示 React 当封装在 `flushSync` 中的代码执行后，立即同步更新 DOM。因此，当你尝试滚动到最后一个待办事项时，它已经在 DOM 中了：