# 20230803-React API(二)-forwardRef

# forwardRef

`forwardRef` 允许你的组件使用 [ref](https://zh-hans.react.dev/learn/manipulating-the-dom-with-refs) 将一个 DOM 节点暴露给父组件。

```js
const SomeComponent = forwardRef(render)
```

## 参考 

### `forwardRef(render)` 

使用 `forwardRef()` 来让你的组件接收一个 ref 并将其传递给一个子组件：

```js
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  // ...
});
```

#### 参数 

- `render`：组件的渲染函数。React 会调用该函数并传入父组件传递来的参数和 `ref`。返回的 JSX 将成为组件的输出。

#### 返回值 

`forwardRef` 返回一个可以在 JSX 中渲染的 React 组件。与作为纯函数定义的 React 组件不同，`forwardRef` 返回的组件还能够接收 `ref` 属性。

### `render` 函数 

`forwardRef` 接受一个渲染函数作为参数。React 将会使用 `props` 和 `ref` 调用此函数：

```jsx
const MyInput = forwardRef(function MyInput(props, ref) {
  return (
    <label>
      {props.label}
      <input ref={ref} />
    </label>
  );
});
```

#### 参数 

- `props`：父组件传递过来的参数。
- `ref`：父组件传递的 `ref` 属性。`ref` 可以是一个对象或函数。如果父组件没有传递一个 ref，那么它将会是 `null`。你应该将接收到的 `ref` 转发给另一个组件，或者将其传递给 [`useImperativeHandle`](https://zh-hans.react.dev/reference/react/useImperativeHandle)。

#### 返回值 

`forwardRef` 返回一个可以在 JSX 中渲染的 React 组件。与作为纯函数定义的 React 组件不同，`forwardRef` 返回的组件还能够接收 `ref` 属性。

## 用法 

### 将 DOM 节点暴露给父组件 

默认情况下，每个组件的 DOM 节点都是私有的。然而，有时候将 DOM 节点公开给父组件是很有用的，比如允许对它进行聚焦。如果你想将其公开，可以将组件定义包装在 `forwardRef()` 中：

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} />
    </label>
  );
});
```

你将在 props 之后收到一个 ref 作为第二个参数。将其传递到要公开的 DOM 节点中：

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} ref={ref} />
    </label>
  );
});
```

这样，父级的 `Form` 组件就能够访问 `MyInput` 暴露的 **`<input>` DOM 节点**：

```jsx
function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <MyInput label="Enter your name:" ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

该 `Form` 组件 [将 ref 传递至](https://zh-hans.react.dev/reference/react/useRef#manipulating-the-dom-with-a-ref) `MyInput`。`MyInput` 组件将该 ref **转发** 至 `<input>` 浏览器标签。因此，`Form` 组件可以访问该 `<input>` DOM 节点并对其调用 [`focus()`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLElement/focus)。

请记住，将组件内部的 ref 暴露给 DOM 节点会使得在稍后更改组件内部更加困难。通常，你会将 DOM 节点从可重用的低级组件中暴露出来，例如按钮或文本输入框，但不会在应用程序级别的组件中这样做，例如头像或评论。

### 在多个组件中转发 ref 

除了将 `ref` 转发到 DOM 节点外，你还可以将其转发到你自己的组件，例如 `MyInput` 组件：

```jsx
const FormField = forwardRef(function FormField(props, ref) {
  // ...
  return (
    <>
      <MyInput ref={ref} />
      ...
    </>
  );
});
```

如果 `MyInput` 组件将 ref 转发给它的 `<input>`，那么 `FormField` 的 ref 将会获得该 `<input>`：

```jsx
function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <FormField label="Enter your name:" ref={ref} isRequired={true} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

`Form` 组件定义了一个 ref 并将其传递给 `FormField`。`FormField` 组件将该 ref 转发给 `MyInput`，后者又将其转发给浏览器的 `<input>` DOM 节点。这就是 `Form` 获取该 DOM 节点的方式。

<iframe src="https://codesandbox.io/embed/hopeful-jepsen-crs8qt?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="hopeful-jepsen-crs8qt"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 暴露一个命令式句柄而不是 DOM 节点 

可以使用一个被称为 **命令式句柄（imperative handle）** 的自定义对象来暴露一个更加受限制的方法集，而不是暴露整个 DOM 节点。为了实现这个目的，你需要定义一个单独的 ref 来存储 DOM 节点：

```jsx
const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  // ...

  return <input {...props} ref={inputRef} />;
});
```

将收到的 `ref` 传递给 [`useImperativeHandle`](https://zh-hans.react.dev/reference/react/useImperativeHandle) 并指定你想要暴露给 `ref` 的值：

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

如果某个组件得到了 `MyInput` 的 ref，则只会接收到 `{ focus, scrollIntoView }` 对象，而不是整个 DOM 节点。这可以让 DOM 节点暴露的信息限制到最小。

<iframe src="https://codesandbox.io/embed/fervent-jang-mdmrj2?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="fervent-jang-mdmrj2"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 陷阱

> **不要滥用 refs**。只有对于作为 props 无法表达的 **命令式** 行为才应该使用 ref：例如滚动到节点、将焦点放在节点上、触发动画，以及选择文本等等。
>
> **如果可以将某些东西使用 props 表达，那就不应该使用 ref**。例如，不要从一个 Modal 组件中暴露一个命令式的句柄，如 `{ open, close }`，更好的做法是像这样使用 `isOpen` 作为一个像 `<Modal isOpen={isOpen} />` 的 prop。[Effects](https://zh-hans.react.dev/learn/synchronizing-with-effects) 可以帮助你通过 props 暴露命令式行为。