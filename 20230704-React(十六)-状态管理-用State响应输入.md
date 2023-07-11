# 20230704-React(十六)-状态管理-用State响应输入

React 控制 UI 的方式是声明式的。你不必直接控制 UI 的各个部分，只需要声明组件可以处于的不同状态，并根据用户的输入在它们之间切换。这与设计师对 UI 的思考方式很相似。

> ### 你将会学习到
>
> - 了解声明式 UI 编程与命令式 UI 编程有何不同
> - 了解如何列举组件可能处于的不同视图状态
> - 了解如何在代码中触发不同视图状态的变化

## 声明式 UI 与命令式 UI 的比较 

当你设计 UI 交互时，可能会去思考 UI 如何根据用户的操作而响应**变化**。想象一个允许用户提交一个答案的表单：

- 当你向表单输入数据时，“提交”按钮会随之变成**可用状态**
- 当你点击“提交”后，表单和提交按钮都会随之变成**不可用状态**，并且会加载动画会随之**出现**
- 如果网络请求成功，表单会随之**隐藏**，同时“提交成功”的信息会随之**出现**
- 如果网络请求失败，错误信息会随之**出现**，同时表单又变为**可用状态**

在 **命令式编程** 中，以上的过程直接告诉你如何去实现交互。你必须去根据要发生的事情写一些明确的命令去操作 UI。对此有另一种理解方式，想象一下，当你坐在车里的某个人旁边，然后一步一步地告诉他该去哪。

![看起来很焦急的司机代表 JavaScript，乘客命令司机执行一系列复杂的转弯导航。](https://zh-hans.react.dev/images/docs/illustrations/i_imperative-ui-programming.png)

他并不知道你想去哪，只想跟着命令行动。（并且如果你发出了错误的命令，那么你就会到达错误的地方）正因为你必须从加载动画到按钮地“命令”每个元素，所以这种告诉计算机**如何**去更新 UI 的编程方式被称为**命令式编程**

在这个命令式 UI 编程的例子中，表单**没有使用** React 生成，而是使用原生的 [DOM](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model):

<iframe src="https://codesandbox.io/embed/practical-kepler-wh03mi?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="practical-kepler-wh03mi"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

对于独立系统来说，命令式地控制用户界面的效果也不错，但是当处于更加复杂的系统中时，这会造成管理的困难程度指数级地增长。如同示例一样，想象一下，当你想更新这样一个包含着不同表单的页面时，你想要添加一个新 UI 元素或一个新的交互，为了保证不会因此产生新的 bug（例如忘记去显示或隐藏一些东西），你必须十分小心地去检查所有已经写好的代码。

React 正是为了解决这样的问题而诞生的。

在 React 中，你不必直接去操作 UI —— 你不必直接启用、关闭、显示或隐藏组件。相反，你只需要 **声明你想要显示的内容，** React 就会通过计算得出该如何去更新 UI。想象一下，当你上了一辆出租车并且告诉司机你想去哪，而不是事无巨细地告诉他该如何走。将你带到目的地是司机的工作，他们甚至可能知道一些你没有想过并且不知道的捷径！

![React 开着车，一个乘客想要去地图上的一个特定的地方。React 会思考如何带你到目的地。](https://zh-hans.react.dev/images/docs/illustrations/i_declarative-ui-programming.png)

## 声明式地考虑 UI 

你已经从上面的例子看到如何去实现一个表单了，为了更好地理解如何在 React 中思考，接下来你将会学到如何用 React 重新实现这个 UI：

1. **定位**你的组件中不同的视图状态
2. **确定**是什么触发了这些 state 的改变
3. **表示**内存中的 state（需要使用 `useState`）
4. **删除**任何不必要的 state 变量
5. **连接**事件处理函数去设置 state

## 步骤 1：定位组件中不同的视图状态 

在计算机科学中，你或许听过可处于多种“状态”之一的 [“状态机”](https://en.wikipedia.org/wiki/Finite-state_machine)。如果你有与设计师一起工作，那么你可能已经见过不同“视图状态”的模拟图。正因为 React 站在设计与计算机科学的交点上，因此这两种思想都是灵感的来源。

首先，你需要去可视化 UI 界面中用户可能看到的所有不同的“状态”：

- **无数据**：表单有一个不可用状态的“提交”按钮。
- **输入中**：表单有一个可用状态的“提交”按钮。
- **提交中**：表单完全处于不可用状态，加载动画出现。
- **成功时**：显示“成功”的消息而非表单。
- **错误时**：与输入状态类似，但会多错误的消息。

像一个设计师一样，你会想要在你添加逻辑之前去“模拟”不同的状态或创建“模拟状态”。例如下面的例子，这是一个对表单可视部分的模拟。这个模拟被一个 `status` 的属性控制，并且这个属性的默认值为 `empty`。

你可以随意命名这个属性，名字并不重要。试着将 `status = 'empty'` 改为 `status = 'success'`，然后你就会看到成功的信息出现。模拟可以让你在书写逻辑前快速迭代 UI。这是同一组件的一个更加充实的原型，仍然由 `status` 属性“控制”：

<iframe src="https://codesandbox.io/embed/determined-jones-pmnojw?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="determined-jones-pmnojw"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

## 深入探讨：同时展示大量的视图状态

如果一个组件有多个视图状态，你可以很方便地将它们展示在一个页面中：

<iframe src="https://codesandbox.io/embed/peaceful-leaf-f0p7sj?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="peaceful-leaf-f0p7sj"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

## 步骤 2：确定是什么触发了这些状态的改变 

你可以触发 state 的更新来响应两种输入：

- **人为**输入。比如点击按钮、在表单中输入内容，或导航到链接。
- **计算机**输入。比如网络请求得到反馈、定时器被触发，或加载一张图片。

![image-20230710175908948](https://s2.loli.net/2023/07/10/izalTuO4AUSh8xf.png)

以上两种情况中，**你必须设置 [state 变量](https://zh-hans.react.dev/learn/state-a-components-memory#anatomy-of-usestate) 去更新 UI**。对于正在开发中的表单来说，你需要改变 state 以响应几个不同的输入：

- **改变输入框中的文本时**（人为）应该根据输入框的内容是否是**空值**，从而决定将表单的状态从空值状态切换到**输入中**或切换回原状态。
- **点击提交按钮时**（人为）应该将表单的状态切换到**提交中**的状态。
- **网络请求成功后**（计算机）应该将表单的状态切换到**成功**的状态。
- **网络请求失败后**（计算机）应该将表单的状态切换到**失败**的状态，与此同时，显示错误信息。

> ### 注意
>
> 注意，人为输入通常需要 [事件处理函数](https://zh-hans.react.dev/learn/responding-to-events)！

为了可视化这个流程，请尝试在纸上画出圆形标签以表示每个状态，两个状态之间的改变用箭头表示。你可以像这样画出很多流程并且在写代码前解决许多 bug。

![流程图从左到右共有 5 个节点。第一个 'empty' 节点通过 'start typing' 分支与 'typing' 节点相连。'typing' 节点则通过 'press submit' 分支与拥有两个分支的 'submitting' 节点相连。其中 'submitting' 节点左侧通过 'network error' 分支与 'error' 节点相连，而右侧则是通过 'network success' 分支与 'success' 节点相连。](https://s2.loli.net/2023/07/10/yKxJuRpSo7f5LVF.png)

## 步骤 3：通过 `useState` 表示内存中的 state

接下来你会需要在内存中通过 [`useState`](https://zh-hans.react.dev/reference/react/useState) 表示组件中的视图状态。诀窍很简单：state 的每个部分都是“处于变化中的”，并且**你需要让“变化的部分”尽可能的少**。更复杂的程序会产生更多 bug！

先从**绝对必须**存在的状态开始。例如，你需要存储输入的 `answer` 以及用于存储最后一个错误的 `error` （如果存在的话）：

```js
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
```

接下来，你需要一个状态变量来代表你想要显示的那个可视状态。通常有多种方式在内存中表示它，因此你需要进行实验。

如果你很难立即想出最好的办法，那就先从添加足够多的 state 开始，**确保**所有可能的视图状态都囊括其中：

```js
const [isEmpty, setIsEmpty] = useState(true);
const [isTyping, setIsTyping] = useState(false);
const [isSubmitting, setIsSubmitting] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [isError, setIsError] = useState(false);
```

你最初的想法或许不是最好的，但是没关系，重构 state 也是步骤中的一部分！

## 步骤 4：删除任何不必要的 state 变量 

你会想要避免 state 内容中的重复，从而只需要关注那些必要的部分。花一点时间来重构你的 state 结构，会让你的组件更容易被理解，减少重复并且避免歧义。你的目的是**防止出现在内存中的 state 不代表任何你希望用户看到的有效 UI 的情况。**（比如你绝对不会想要在展示错误信息的同时禁用掉输入框，导致用户无法纠正错误！）

这有一些你可以问自己的， 关于 state 变量的问题：

- **这个 state 是否会导致矛盾**？例如，`isTyping` 与 `isSubmitting` 的状态不能同时为 `true`。矛盾的产生通常说明了这个 state 没有足够的约束条件。两个布尔值有四种可能的组合，但是只有三种对应有效的状态。为了将“不可能”的状态移除，你可以将 `'typing'`、`'submitting'` 以及 `'success'` 这三个中的其中一个与 `status` 结合。
- **相同的信息是否已经在另一个 state 变量中存在**？另一个矛盾：`isEmpty` 和 `isTyping` 不能同时为 `true`。通过使它们成为独立的 state 变量，可能会导致它们不同步并导致 bug。幸运的是，你可以移除 `isEmpty` 转而用 `message.length === 0`。
- **你是否可以通过另一个 state 变量的相反值得到相同的信息**？`isError` 是多余的，因为你可以检查 `error !== null`。

在清理之后，你只剩下 3 个（从原本的 7 个！）*必要*的 state 变量：

```js
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [status, setStatus] = useState('typing'); // 'typing', 'submitting', or 'success'
```

正是因为你不能在不破坏功能的情况下删除其中任何一个状态变量，因此你可以确定这些都是必要的。

### 深入探讨：通过 reducer 来减少“不可能” state

尽管这三个变量对于表示这个表单的状态来说已经足够好了，仍然是有一些中间状态并不是完全有意义的。例如一个非空的 `error` 当 `status` 的值为 `success` 时没有意义。为了更精确地模块化状态，你可以 [将状态提取到一个 reducer 中](https://zh-hans.react.dev/learn/extracting-state-logic-into-a-reducer)。Reducer 可以让您合并多个状态变量到一个对象中并巩固所有相关的逻辑！

## 步骤 5：连接事件处理函数以设置 state

最后，创建事件处理函数去设置 state 变量。下面是绑定好事件的最终表单：

<iframe src="https://codesandbox.io/embed/gallant-sun-vpttuv?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="gallant-sun-vpttuv"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

尽管这些代码相对与最初的命令式的例子来说更长，但是却更加健壮。将所有的交互变为 state 的改变，可以让你避免之后引入新的视图状态后导致现有 state 被破坏。同时也使你在不必改变交互逻辑的情况下，更改每个状态对应的 UI。

## 摘要

- 声明式编程意味着为每个视图状态声明 UI 而非细致地控制 UI（命令式）。
- 当开发一个组件时：
  1. 写出你的组件中所有的视图状态。
  2. 确定是什么触发了这些 state 的改变。
  3. 通过 `useState` 模块化内存中的 state。
  4. 删除任何不必要的 state 变量。
  5. 连接事件处理函数去设置 state。
