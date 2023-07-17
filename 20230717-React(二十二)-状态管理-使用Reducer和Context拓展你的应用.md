# 20230717-React(二十二)-状态管理-使用Reducer和Context拓展你的应用

Reducer 可以整合组件的状态更新逻辑。Context 可以将信息深入传递给其他组件。你可以组合使用它们来共同管理一个复杂页面的状态。

> ### 你将会学习到
>
> - 如何结合使用 reducer 和 context
> - 如何去避免通过 props 传递 state 和 dispatch
> - 如何将 context 和状态逻辑保存在一个单独的文件中

## 结合使用 reducer 和 context 

在 [reducer 介绍](https://zh-hans.react.dev/learn/extracting-state-logic-into-a-reducer) 的例子里面，状态被 reducer 所管理。reducer 函数包含了所有的状态更新逻辑并在此文件的底部声明：

<iframe src="https://codesandbox.io/embed/bold-rui-3lvffx?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="bold-rui-3lvffx"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

Reducer 有助于保持事件处理程序的简短明了。但随着应用规模越来越庞大，你就可能会遇到别的困难。**目前，`tasks` 状态和 `dispatch` 函数仅在顶级 `TaskApp` 组件中可用**。要让其他组件读取任务列表或更改它，你必须显式 [传递](https://zh-hans.react.dev/learn/passing-props-to-a-component) 当前状态和事件处理程序，将其作为 props。

例如，`TaskApp` 将 一系列 task 和事件处理程序传递给 `TaskList`：

```jsx
<TaskList
  tasks={tasks}
  onChangeTask={handleChangeTask}
  onDeleteTask={handleDeleteTask}
/>
```

`TaskList` 将事件处理程序传递给 `Task`：

```jsx
<Task
  task={task}
  onChange={onChangeTask}
  onDelete={onDeleteTask}
/>
```

在像这样的小示例里这样做没什么问题，但是如果你有成千上百个组件，传递所有状态和函数可能会非常麻烦！

这就是为什么，比起通过 props 传递它们，你可能想把 `tasks` 状态和 `dispatch` 函数都 [放入 context](https://zh-hans.react.dev/learn/passing-data-deeply-with-context)。**这样，所有的在 `TaskApp` 组件树之下的组件都不必一直往下传 props 而可以直接读取 tasks 和 dispatch 函数**。

下面将介绍如何结合使用 reducer 和 context：

1. **创建** context。
2. 将 state 和 dispatch **放入** context。
3. 在组件树的任何地方 **使用** context。

### 第一步: 创建 context 

`useReducer` 返回当前的 `tasks` 和 `dispatch` 函数来让你更新它们：

```js
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

为了将它们从组件树往下传，你将 [创建](https://zh-hans.react.dev/learn/passing-data-deeply-with-context#step-2-use-the-context) 两个不同的 context：

- `TasksContext` 提供当前的 tasks 列表。
- `TasksDispatchContext` 提供了一个函数可以让组件分发动作。

将它们从单独的文件导出，以便以后可以从其他文件导入它们：

<iframe src="https://codesandbox.io/embed/unruffled-cookies-yfomxd?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="unruffled-cookies-yfomxd"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

在这里，你把 `null` 作为默认值传递给两个 context。实际值是由 `TaskApp` 组件提供的。

### 第二步: 将 state 和 dispatch 函数 放入 context 

现在，你可以将所有的 context 导入 `TaskApp` 组件。获取 `useReducer()` 返回的 `tasks` 和 `dispatch` 并将它们 [提供](https://zh-hans.react.dev/learn/passing-data-deeply-with-context#step-3-provide-the-context) 给整个组件树：

```jsx
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  // ...
  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        ...
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

现在，你可以同时通过 props 和 context 传递信息：

<iframe src="https://codesandbox.io/embed/condescending-brattain-gbluz5?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="condescending-brattain-gbluz5"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

在下一步中，你将删除通过 props 传递的代码。

### Step 3: 在组件树中的任何地方使用 context 

现在你不需要将 tasks 和事件处理程序在组件树中传递：

```jsx
<TasksContext.Provider value={tasks}>
  <TasksDispatchContext.Provider value={dispatch}>
    <h1>Day off in Kyoto</h1>
    <AddTask />
    <TaskList />
  </TasksDispatchContext.Provider>
</TasksContext.Provider>
```

相反，任何需要 tasks 的组件都可以从 `TaskContext` 中读取它：

```js
export default function TaskList() {
  const tasks = useContext(TasksContext);
  // ...
```

任何组件都可以从 context 中读取 `dispatch` 函数并调用它，从而更新任务列表：

```jsx
export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext);
  // ...
  return (
    // ...
    <button onClick={() => {
      setText('');
      dispatch({
        type: 'added',
        id: nextId++,
        text: text,
      });
    }}>Add</button>
    // ...
```

**`TaskApp` 组件不会向下传递任何事件处理程序，`TaskList` 也不会**。每个组件都会读取它需要的 context：

<iframe src="https://codesandbox.io/embed/flamboyant-pasteur-v4f3rx?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="flamboyant-pasteur-v4f3rx"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

**state 仍然 “存在于” 顶层 `Task` 组件中，由 `useReducer` 进行管理**。不过，组件树里的组件只要导入这些 context 之后就可以获取 `tasks` 和 `dispatch`。

## 将相关逻辑迁移到一个文件当中 

这不是必须的，但你可以通过将 reducer 和 context 移动到单个文件中来进一步整理组件。目前，“TasksContext.js” 仅包含两个 context 声明：

```js
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

来给这个文件添加更多代码！将 reducer 移动到此文件中，然后声明一个新的 `TasksProvider` 组件。此组件将所有部分连接在一起：

1. 它将管理 reducer 的状态。
2. 它将提供现有的 context 给组件树。
3. 它将 [把 `children` 作为 prop](https://zh-hans.react.dev/learn/passing-props-to-a-component#passing-jsx-as-children)，所以你可以传递 JSX。

```jsx
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

**这将使 `TaskApp` 组件更加直观：**

<iframe src="https://codesandbox.io/embed/divine-snowflake-2ro1iu?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="divine-snowflake-2ro1iu"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

你也可以从 `TasksContext.js` 中导出使用 context 的函数：

```js
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}
```

组件可以通过以下函数读取 context：

```js
const tasks = useTasks();
const dispatch = useTasksDispatch();
```

这不会改变任何行为，但它会允许你之后进一步分割这些 context 或向这些函数添加一些逻辑。**现在所有的 context 和 reducer 连接部分都在 `TasksContext.js` 中。这保持了组件的干净和整洁，让我们专注于它们显示的内容，而不是它们从哪里获得数据：**

<iframe src="https://codesandbox.io/embed/goofy-hooks-ww49ef?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="goofy-hooks-ww49ef"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

你可以将 `TasksProvider` 视为页面的一部分，它知道如何处理 tasks。`useTasks` 用来读取它们，`useTasksDispatch` 用来从组件树下的任何组件更新它们。

> ### 注意
>
> 像 `useTasks` 和 `useTasksDispatch` 这样的函数被称为 **[自定义 Hook](https://zh-hans.react.dev/learn/reusing-logic-with-custom-hooks)。** 如果你的函数名以 `use` 开头，它就被认为是一个自定义 Hook。这让你可以使用其他 Hook，比如 `useContext`。

随着应用的增长，你可能会有许多这样的 context 和 reducer 的组合。这是一种强大的拓展应用并 [提升状态](https://zh-hans.react.dev/learn/sharing-state-between-components) 的方式，让你在组件树深处访问数据时无需进行太多工作。

## 摘要

- 你可以将 reducer 与 context 相结合，让任何组件读取和更新它的状态。
- 为子组件提供 state 和 dispatch 函数：
  1. 创建两个 context (一个用于 state,一个用于 dispatch 函数)。
  2. 让组件的 context 使用 reducer。
  3. 使用组件中需要读取的 context。
- 你可以通过将所有传递信息的代码移动到单个文件中来进一步整理组件。
  - 你可以导出一个像 `TasksProvider` 可以提供 context 的组件。
  - 你也可以导出像 `useTasks` 和 `useTasksDispatch` 这样的自定义 Hook。
- 你可以在你的应用程序中大量使用 context 和 reducer 的组合。