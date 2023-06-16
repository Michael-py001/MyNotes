# 20230615-React学习(十)-添加交互-state: 组件的记忆

组件通常需要根据交互更改屏幕上显示的内容。输入表单应该更新输入字段，单击轮播图上的“下一个”应该更改显示的图片，单击“购买”应该将商品放入购物车。组件需要“记住”某些东西：当前输入值、当前图片、购物车。在 React 中，这种组件特有的记忆被称为 **state**。

> ### 你将会学习到
>
> - 如何使用 [`useState`](https://zh-hans.react.dev/reference/react/useState) Hook 添加 state 变量
> - `useState` Hook 返回哪一对值
> - 如何添加多个 state 变量
> - 为什么 state 被称作是局部的

以下是一个渲染雕塑图片的组件。点击 “Next” 按钮应该显示下一个雕塑并将 `index` 更改为 `1`，再次点击又更改为 `2`，以此类推。但这个组件现在**不起作用**（你可以试一试！）：

<iframe src="https://codesandbox.io/embed/compassionate-chaplygin-tejhuy?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="compassionate-chaplygin-tejhuy"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

`handleClick()` 事件处理函数正在更新局部变量 `index`。但存在两个原因使得变化不可见：

1. **局部变量无法在多次渲染中持久保存。** 当 React 再次渲染这个组件时，它会从头开始渲染——不会考虑之前对局部变量的任何更改。
2. **更改局部变量不会触发渲染。** React 没有意识到它需要使用新数据再次渲染组件。

要使用新数据更新组件，需要做两件事：

1. **保留** 渲染之间的数据。
2. **触发** React 使用新数据渲染组件（重新渲染）。

[`useState`](https://zh-hans.react.dev/reference/react/useState) Hook 提供了这两个功能：

1. **State 变量** 用于保存渲染间的数据。
2. **State setter 函数** 更新变量并触发 React 再次渲染组件。

## 添加一个 state 变量 

要添加 state 变量，先从文件顶部的 React 中导入 `useState`：

```js
import { useState } from 'react';
```

然后，替换这一行：

```js
let index = 0;
```

将其修改为

```js
const [index, setIndex] = useState(0);
```

`index` 是一个 state 变量，`setIndex` 是对应的 setter 函数。

> 这里的 `[` 和 `]` 语法称为[数组解构](https://zh-hans.react.dev/learn/a-javascript-refresher#array-destructuring)，它允许你从数组中读取值。 `useState` 返回的数组总是正好有两项。

以下展示了它们在 `handleClick()` 中是如何共同起作用的：

```js
function handleClick() {
  setIndex(index + 1);
}
```

现在点击 “Next” 按钮切换当前雕塑：

<iframe src="https://codesandbox.io/embed/upbeat-keldysh-0efc0f?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="upbeat-keldysh-0efc0f"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 遇见你的第一个 Hook 

在 React 中，`useState` 以及任何其他以“`use`”开头的函数都被称为 **Hook**。

Hook 是特殊的函数，只在 React [渲染](https://zh-hans.react.dev/learn/render-and-commit#step-1-trigger-a-render)时有效（我们将在下一节详细介绍）。它们能让你 “hook” 到不同的 React 特性中去。

State 只是这些特性中的一个，你之后还会遇到其他 Hook。

> ### **陷阱**
>
> **Hooks ——以 `use` 开头的函数——只能在组件或[自定义 Hook](https://zh-hans.react.dev/learn/reusing-logic-with-custom-hooks) 的最顶层调用。** 你不能在条件语句、循环语句或其他嵌套函数内调用 Hook。Hook 是函数，但将它们视为关于组件需求的无条件声明会很有帮助。在组件顶部 “use” React 特性，类似于在文件顶部“导入”模块。

### 剖析 `useState` 

当你调用 [`useState`](https://zh-hans.react.dev/reference/react/useState) 时，你是在告诉 React 你想让这个组件记住一些东西：

```
const [index, setIndex] = useState(0);
```

在这个例子里，你希望 React 记住 `index`。

> ### **注意**
>
> 惯例是将这对返回值命名为 `const [thing, setThing]`。你也可以将其命名为任何你喜欢的名称，但遵照约定俗成能使跨项目合作更易理解。

`useState` 的唯一参数是 state 变量的**初始值**。在这个例子中，`index` 的初始值被`useState(0)`设置为 `0`。

每次你的组件渲染时，`useState` 都会给你一个包含两个值的数组：

1. **state 变量** (`index`) 会保存上次渲染的值。
2. **state setter 函数** (`setIndex`) 可以更新 state 变量并触发 React 重新渲染组件。

以下是实际发生的情况：

```ts
const [index, setIndex] = useState(0);
```

1. **组件进行第一次渲染。** 因为你将 `0` 作为 `index` 的初始值传递给 `useState`，它将返回 `[0, setIndex]`。 React 记住 `0` 是最新的 state 值。
2. **你更新了 state**。当用户点击按钮时，它会调用 `setIndex(index + 1)`。 `index` 是 `0`，所以它是 `setIndex(1)`。这告诉 React 现在记住 `index` 是 `1` 并触发下一次渲染。
3. **组件进行第二次渲染**。React 仍然看到 `useState(0)`，但是因为 React *记住* 了你将 `index` 设置为了 `1`，它将返回 `[1, setIndex]`。
4. 以此类推！



> 也就是说传入`useState`的初始值是不会变的，即使传入的是个变量，改变的只有它内部返回的值。
>
> ```js
> let num = 0
> const [index, setIndex] = useState(num);
> 
> setIndex(index + 1)  //只会改变index 不会改变num
> ```

## 赋予一个组件多个 state 变量 

你可以在一个组件中拥有任意多种类型的 state 变量。该组件有两个 state 变量，一个数字 `index` 和一个布尔值 `showMore`，点击 “Show Details” 会改变 `showMore` 的值：

```js
const [index, setIndex] = useState(0);
const [showMore, setShowMore] = useState(false);
```

<iframe src="https://codesandbox.io/embed/agitated-burnell-jcnbsj?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="agitated-burnell-jcnbsj"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

如果它们不相关，那么存在多个 state 变量是一个好主意，例如本例中的 `index` 和 `showMore`。但是，如果你发现经常同时更改两个 state 变量，那么最好将它们合并为一个。例如，如果你有一个包含多个字段的表单，那么有一个值为对象的 state 变量比每个字段对应一个 state 变量更方便。 [选择 state 结构](https://zh-hans.react.dev/learn/choosing-the-state-structure)在这方面有更多提示。

> ##### 深入探讨: React 如何知道返回哪个 state 
>
> 你可能已经注意到，`useState` 在调用时没有任何关于它引用的是*哪个* state 变量的信息。没有传递给 `useState` 的“标识符”，它是如何知道要返回哪个 state 变量呢？它是否依赖于解析函数之类的魔法？答案是否定的。
>
> 相反，为了使语法更简洁，**在同一组件的每次渲染中，Hooks 都依托于一个稳定的调用顺序**。这在实践中很有效，因为如果你遵循上面的规则（“只在顶层调用 Hooks”），Hooks 将始终以相同的顺序被调用。此外，[linter 插件](https://www.npmjs.com/package/eslint-plugin-react-hooks)也可以捕获大多数错误。
>
> 在 React 内部，为每个组件保存了一个数组，其中每一项都是一个 state 对。它维护当前 state 对的索引值，在渲染之前将其设置为 “0”。每次调用 useState 时，React 都会为你提供一个 state 对并增加索引值。你可以在文章 [React Hooks: not magic, just arrays](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e)中阅读有关此机制的更多信息。
>
> 这个例子**没有使用 React**，但它让你了解 `useState` 在内部是如何工作的：
>
> <iframe src="https://codesandbox.io/embed/laughing-snow-x6f50i?fontsize=14&hidenavigation=1&theme=dark"
>      style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
>      title="laughing-snow-x6f50i"
>      allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
>      sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
>    ></iframe>
>
> ```js
> let componentHooks = [];
> let currentHookIndex = 0; //专门用来记录useState调用的次数
> 
> // useState 在 React 中是如何工作的（简化版）
> function useState(initialState) {
>   let pair = componentHooks[currentHookIndex];
>   if (pair) {//pair已存在，直接返回当前pair，同时currentHookIndex++，等下一次调用时不存在会走下面的逻辑
>     currentHookIndex++;
>     return pair;
>   }
> 
>   // 这是我们第一次进行渲染
>   // 所以新建一个 state pair 然后存储它
>   pair = [initialState, setState];
> 
>   function setState(nextState) {
>     // 当用户发起 state 的变更，
>     // 把新的值放入 pair 中，为什么是pair[0]呢？原因是useState返回的就是一个数组，数组第一个就是值
>     pair[0] = nextState; 
>     updateDOM();
>   }
> 
>   // 存储这个 pair 用于将来的渲染
>   // 并且为下一次 hook 的调用做准备
>   componentHooks[currentHookIndex] = pair;
>   currentHookIndex++;
>   return pair;
> }
> 
> function Gallery() {
>   // 每次调用 useState() 都会得到新的 pair
>   const [index, setIndex] = useState(0);
>   const [showMore, setShowMore] = useState(false);
> 
>   function handleNextClick() {
>     setIndex(index + 1);
>   }
> 
>   function handleMoreClick() {
>     setShowMore(!showMore);
>   }
> 
>   let sculpture = sculptureList[index];
>   // 这个例子没有使用 React，所以
>   // 返回一个对象而不是 JSX
>   return {
>     onNextClick: handleNextClick,
>     onMoreClick: handleMoreClick,
>     header: `${sculpture.name} by ${sculpture.artist}`,
>     counter: `${index + 1} of ${sculptureList.length}`,
>     more: `${showMore ? 'Hide' : 'Show'} details`,
>     description: showMore ? sculpture.description : null,
>     imageSrc: sculpture.url,
>     imageAlt: sculpture.alt
>   };
> }
> 
> function updateDOM() {
>   // 在渲染组件之前
>   // 重置当前 Hook 的下标
>   currentHookIndex = 0;
>   let output = Gallery();
> 
>   // 更新 DOM 以匹配输出结果
>   // 这部分工作由 React 为你完成
>   nextButton.onclick = output.onNextClick;
>   header.textContent = output.header;
>   moreButton.onclick = output.onMoreClick;
>   moreButton.textContent = output.more;
>   image.src = output.imageSrc;
>   image.alt = output.imageAlt;
>   if (output.description !== null) {
>     description.textContent = output.description;
>     description.style.display = '';
>   } else {
>     description.style.display = 'none';
>   }
> }
> 
> let nextButton = document.getElementById('nextButton');
> let header = document.getElementById('header');
> let moreButton = document.getElementById('moreButton');
> let description = document.getElementById('description');
> let image = document.getElementById('image');
> let sculptureList = [{
>   name: 'Homenaje a la Neurocirugía',
>   artist: 'Marta Colvin Andrade',
>   description: 'Although Colvin is predominantly known for abstract themes that allude to pre-Hispanic symbols, this gigantic sculpture, an homage to neurosurgery, is one of her most recognizable public art pieces.',
>   url: 'https://i.imgur.com/Mx7dA2Y.jpg',
>   alt: 'A bronze statue of two crossed hands delicately holding a human brain in their fingertips.'  
> }, {
>   name: 'Floralis Genérica',
>   artist: 'Eduardo Catalano',
>   description: 'This enormous (75 ft. or 23m) silver flower is located in Buenos Aires. It is designed to move, closing its petals in the evening or when strong winds blow and opening them in the morning.',
>   url: 'https://i.imgur.com/ZF6s192m.jpg',
>   alt: 'A gigantic metallic flower sculpture with reflective mirror-like petals and strong stamens.'
> }];
> 
> // 使 UI 匹配当前 state
> updateDOM();
> 
> ```
>
> 你不必理解它就可以使用 React，但你可能会发现这是一个有用的心智模型。

## State 是隔离且私有的 

State 是屏幕上组件实例内部的状态。换句话说，**如果你渲染同一个组件两次，每个副本都会有完全隔离的 state**！改变其中一个不会影响另一个。

在这个例子中，之前的 `Gallery` 组件以同样的逻辑被渲染了两次。试着点击每个画廊内的按钮。你会注意到它们的 state 是相互独立的：

<iframe src="https://codesandbox.io/embed/divine-currying-rodp3h?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="divine-currying-rodp3h"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

这就是 state 与声明在模块顶部的普通变量不同的原因。 State 不依赖于特定的函数调用或在代码中的位置，它的作用域“只限于”屏幕上的某块特定区域。你渲染了两个 `<Gallery />` 组件，所以它们的 state 是分别存储的。

> **个人理解**：useState()本身是一个函数，而函数每次执行都会产生一个新的上下文，这个上下文中的代码是存储在堆内存中的(引用内存)，在外层函数引用了这其中的变量时，内存不会被回收。
>
> 同时每个组件又是一个函数，所以每个组件渲染都是一次函数的执行，所产生的上下文是相互独立的，所以每个组件都是独立的。

还要注意 `Page` 组件“不知道”关于 `Gallery` state 的任何信息，甚至不知道它是否有任何 state。与 props 不同，**state 完全私有于声明它的组件**。父组件无法更改它。这使你可以向任何组件添加或删除 state，而不会影响其他组件。

如果你希望两个画廊保持其 states 同步怎么办？在 React 中执行此操作的正确方法是从子组件中*删除* state 并将其添加到离它们最近的共享父组件中。接下来的几节将专注于组织单个组件的 state，但我们将在[组件间共享 state](https://zh-hans.react.dev/learn/sharing-state-between-components) 中回到这个主题。

## 摘要

- 当一个组件需要在多次渲染间“记住”某些信息时使用 state 变量。
- State 变量是通过调用 `useState` Hook 来声明的。
- Hook 是以 `use` 开头的特殊函数。它们能让你 “hook” 到像 state 这样的 React 特性中。
- Hook 可能会让你想起 import：它们需要在非条件语句中调用。调用 Hook 时，包括 `useState`，仅在组件或另一个 Hook 的顶层被调用才有效。
- `useState` Hook 返回一对值：当前 state 和更新它的函数。
- 你可以拥有多个 state 变量。在内部，React 按顺序匹配它们。
- State 是组件私有的。如果你在两个地方渲染它，则每个副本都有独属于自己的 state。



> State 变量仅用于在组件重渲染时保存信息。在单个事件处理函数中，普通变量就足够了。当普通变量运行良好时，不要引入 state 变量。比如以下代码：
>
> ```js
> export default function FeedbackForm() {
>   function handleClick() {
>     const name = prompt('What is your name?');
>     alert(`Hello, ${name}!`);
>   }
> 
>   return (
>     <button onClick={handleClick}>
>       Greet
>     </button>
>   );
> }
> ```

