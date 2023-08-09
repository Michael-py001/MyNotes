# 20230809-深入浅出React-React-Hooks设计动机与工作模式(下)

经过第 6 课时的学习，相信你已经清楚了 React-Hooks 的来头，并理解了其背后的“设计动机”。本课时我们的任务是构建对 React-Hooks 的整体认知。

在本课时的主体部分，我将通过一系列的编码实例来帮助你认识 useState、useEffect 这两个有代表性的 Hook，这一步意在帮助初学者对 React-Hooks 可以快速上手。在此基础上，我们将重新理解“Why React-Hooks”这个问题。在课时的最后，我将结合自身的开发体验，和你分享当下这个阶段，我所认识到的 Hooks 的局限性。

> 注：在学习本课时的过程中，请你摒弃“认识的 API 名字越多就越牛”这种错误的学习理念。如果你希望掌握尽可能多的 Hook 的用法，点击这里可以一键进入 React-Hooks API 文档的海洋。对本课时来说，所有涉及对 API 用法的介绍都是 “教具”，仅仅是为后续更深层次的知识讲解作铺垫。

## 先导知识：从核心 API 看 Hooks 的基本形态

### useState()：为函数组件引入状态

早期的函数组件相比于类组件，其一大劣势是缺乏定义和维护 state 的能力，而 state（状态）作为 React 组件的灵魂，必然是不可省略的。因此 React-Hooks 在诞生之初，就优先考虑了对 state 的支持。useState 正是这样一个能够为函数组件引入状态的 API。

### 函数组件，真的很轻

在过去，你可能会为了使用 state，不得不去编写一个类组件（这里我给出一个 Demo，编码如下所示）：

```jsx
import React, { Component } from "react";
export default class TextButton extends Component {

  constructor() {
    super();
    this.state = {
      text: "初始文本"
    };
  }

  changeText = () => {
    this.setState(() => {
      return {
        text: "修改后的文本"
      };
    });
  };
  render() {
    const { text } = this.state;
    return (
      <div className="textButton">
        <p>{text}</p>
        <button onClick={this.changeText}>点击修改文本</button>
      </div>
    );
  }
```

有了 useState 后，我们就可以直接在函数组件里引入 state。以下是使用 useState 改造过后的 TextButton 组件：

```jsx
import React, { useState } from "react";
export default function Button() {
  const [text, setText] = useState("初始文本");
  function changeText() {
    return setText("修改后的文本");
  }
  return (
    <div className="textButton">
      <p>{text}</p>
      <button onClick={changeText}>点击修改文本</button>
    </div>
  );
}
```

上面两套代码实现的界面交互效果完全一样，而函数组件的代码量几乎是类组件代码量的一半！

如果你在第 06 课时曾或多或少地对“类组件太重了”这个观点感到茫然，那么相信眼前这个 Demo 足以让你真真切切地感受到两类组件在复杂度上的差异——同样逻辑的函数组件相比类组件而言，复杂度要低得多得多。

### useState 快速上手

从用法上看，useState 返回的是一个数组，数组的第一个元素对应的是我们想要的那个 state 变量，第二个元素对应的是能够修改这个变量的 API。我们可以通过数组解构的语法，将这两个元素取出来，并且按照我们自己的想法命名。一个典型的调用示例如下：

```js
const [state, setState] = useState(initialState);
```

在这个示例中，我们给自己期望的那个状态变量命名为 state，给修改 state 的 API 命名为 setState。useState 中传入的 initialState 正是 state 的初始值。后续我们可以通过调用 setState，来修改 state 的值，像这样：

```js
setState(newState)
```

状态更新后会触发渲染层面的更新，这点和类组件是一致的。

这里需要向初学者强调的一点是：状态和修改状态的 API 名都是可以自定义的。比如在上文的 Demo 中，就分别将其自定义为 text 和 setText：

```js
const [text, setText] = useState("初始文本");
```

“set + 具体变量名”这种命名形式，可以帮助我们快速地将 API 和它对应的状态建立逻辑联系。

当我们在函数组件中调用 React.useState 的时候，实际上是给这个组件关联了一个状态——注意，是“一个状态”而不是“一批状态”。这一点是相对于类组件中的 state 来说的。在类组件中，我们定义的 state 通常是一个这样的对象，如下所示：

```js
this.state {
  text: "初始文本",
  length: 10000,
  author: ["xiuyan", "cuicui", "yisi"]
}
```

这个对象是“包容万物”的：整个组件的状态都在 state 对象内部做收敛，当我们需要某个具体状态的时候，会通过 this.state.xxx 这样的访问对象属性的形式来读取它。

而在 useState 这个钩子的使用背景下，state 就是单独的一个状态，它可以是任何你需要的 JS 类型。像这样：

```js
// 定义为数组
const [author, setAuthor] = useState(["xiuyan", "cuicui", "yisi"]);
 
// 定义为数值
const [length, setLength] = useState(100);
// 定义为字符串
const [text, setText] = useState("初始文本")
```

你还可以定义为布尔值、对象等，都是没问题的。它就像类组件中 state 对象的某一个属性一样，对应着一个单独的状态，允许你存储任意类型的值。

### useEffect()：允许函数组件执行副作用操作

函数组件相比于类组件来说，最显著的差异就是 state 和生命周期的缺失。useState 为函数组件引入了 state，而 useEffect 则在一定程度上弥补了生命周期的缺席。

useEffect 能够为函数组件引入副作用。过去我们习惯放在 componentDidMount、componentDidUpdate 和 componentWillUnmount 三个生命周期里来做的事，现在可以放在 useEffect 里来做，比如操作 DOM、订阅事件、调用外部 API 获取数据等。

### useEffect 和生命周期函数之间的“替换”关系

我们可以通过下面这个例子来理解 useEffect 和生命周期函数之间的替换关系。这里我先给到你一个用 useEffect 编写的函数组件示例：

```JSX
// 注意 hook 在使用之前需要引入
import React, { useState, useEffect } from 'react';
// 定义函数组件
function IncreasingTodoList() {
  // 创建 count 状态及其对应的状态修改函数
  const [count, setCount] = useState(0);
  // 此处的定位与 componentDidMount 和 componentDidUpdate 相似
  useEffect(() => {
    // 每次 count 增加时，都增加对应的待办项
    const todoList = document.getElementById("todoList");
    const newItem = document.createElement("li");
    newItem.innerHTML = `我是第${count}个待办项`;
    todoList.append(newItem);
  });
  // 编写 UI 逻辑
  return (
    <div>
      <p>当前共计 {count} 个todo Item</p>
      <ul id="todoList"></ul>
      <button onClick={() => setCount(count + 1)}>点我增加一个待办项</button>
    </div>
  );
}
```

通过上面这段代码构造出来的界面在刚刚挂载完毕时，就是如下的样子：

<iframe src="https://codesandbox.io/embed/suspicious-surf-88lr8k?fontsize=14&hidenavigation=1&module=%2Fsrc%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="suspicious-surf-88lr8k"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

IncreasingTodoList 是一个只允许增加 item 的 ToDoList（待办事项列表）。按照 useEffect 的设定，每当我们点击“点我增加一个待办项”这个按钮，驱动 count+1 的同时，DOM 结构里也会被追加一个 li 元素。以下是连击按钮三次之后的效果图：

![image-20230809114549434](https://s2.loli.net/2023/08/09/PteKdoUqmHCJvLY.png)

同样的效果，按照注释里的提示，我们也可以通过编写 class 组件来实现：

```jsx
import React from 'react';
// 定义类组件
class IncreasingTodoList extends React.Component {

  // 初始化 state
  state = { count: 0 }
  // 此处调用上个 demo 中 useEffect 中传入的函数
  componentDidMount() {
    this.addTodoItem()
  }
  // 此处调用上个 demo 中 useEffect 中传入的函数
  componentDidUpdate() {
    this.addTodoItem()
  }
  // 每次 count 增加时，都增加对应的待办项
  addTodoItem = () => {
    const { count } = this.state
    const todoList = document.getElementById("todoList")
    const newItem = document.createElement("li")
    newItem.innerHTML = `我是第${count}个待办项`
    todoList.append(newItem)
  }

  // 定义渲染内容
  render() {
    const { count } = this.state
    return (
      <div>
        <p>当前共计 {count} 个todo Item</p>
        <ul id="todoList"></ul>
        <button
          onClick={() =>
            this.setState({
              count: this.state.count + 1,
            })
          }
        >
          点我增加一个待办项
        </button>
      </div>
    )
  }
}
```

通过这样一个对比，类组件生命周期和函数组件 useEffect 之间的转换关系可以说是跃然纸上了。

在这里，我提个醒：初学 useEffect 时，我们难免习惯于借助对生命周期的理解来推导对 useEffect 的理解。但长期来看，若是执着于这个学习路径，无疑将阻碍你真正从心智模式的层面拥抱 React-Hooks。

**有时候，我们必须学会忘记旧的知识，才能够更好地拥抱新的知识。**对于每一个学习 useEffect 的人来说，生命周期到 useEffect 之间的转换关系都不是最重要的，最重要的是在脑海中构建一个“组件有副作用 → 引入 useEffect”这样的条件反射——**当你真正抛却类组件带给你的刻板印象、拥抱函数式编程之后，想必你会更加认同“useEffect 是用于为函数组件引入副作用的钩子”这个定义。**

### useEffect 快速上手

useEffect 可以接收两个参数，分别是回调函数与依赖数组，如下面代码所示：

```js
useEffect(callBack, [])
```

useEffect 用什么姿势来调用，本质上取决于你想用它来达成什么样的效果。下面我们就以效果为线索，简单介绍 useEffect 的调用规则。

- 每一次渲染后都执行的副作用：传入回调函数，不传依赖数组。调用形式如下所示：

  ```js
  useEffect(callBack)
  ```

- 仅在挂载阶段执行一次的副作用：传入回调函数，且这个函数的返回值不是一个函数，同时传入一个空数组。调用形式如下所示：

  ```js
  useEffect(()=>{
    // 这里是业务逻辑 
  }, [])
  ```

- 仅在挂载阶段和卸载阶段执行的副作用：传入回调函数，且这个函数的返回值是一个函数，同时传入一个空数组。假如回调函数本身记为 A， 返回的函数记为 B，那么将在挂载阶段执行 A，卸载阶段执行 B。调用形式如下所示：

  ```js
  useEffect(()=>{
    // 这里是 A 的业务逻辑
  
    // 返回一个函数记为 B
    return ()=>{
    }
  }, [])
  ```

  这里需要注意，这种调用方式之所以会在卸载阶段去触发 B 函数的逻辑，是由 useEffect 的执行规则决定的：**useEffect 回调中返回的函数被称为“清除函数”**，当 React 识别到清除函数时，会在调用新的 effect 逻辑之前执行清除函数内部的逻辑。**这个规律不会受第二个参数或者其他因素的影响，只要你在 useEffect 回调中返回了一个函数，它就会被作为清除函数来处理**。

- 每一次渲染都触发，且卸载阶段也会被触发的副作用：传入回调函数，且这个函数的返回值是一个函数，同时不传第二个参数。如下所示：

  ```js
  useEffect(()=>{
    // 这里是 A 的业务逻辑
  
    // 返回一个函数记为 B
    return ()=>{
    }
  })
  ```

上面这段代码就会使得 React 在每一次渲染都去触发 A 逻辑，并且在下一次 A 逻辑被触发之前去触发 B 逻辑。

其实你只要记住，如果你有一段 effect 逻辑，需要在每次调用它之前对上一次的 effect 进行清理，那么把对应的清理逻辑写进 useEffect 回调的返回函数（上面示例中的 B 函数）里就行了。

- 根据一定的依赖条件来触发的副作用：传入回调函数，同时传入一个非空的数组，如下所示

  ```js
  useEffect(()=>{
    // 这是回调函数的业务逻辑 
  
    // 若 xxx 是一个函数，则 xxx 会在组件每次因 num1、num2、num3 的改变而重新渲染时被触发
    return xxx
  }, [num1, num2, num3])
  ```

这里我给出的一个示意数组是 [num1, num2, num3]。首先需要说明，数组中的变量一般都是来源于组件本身的数据（props 或者 state）。若数组不为空，那么 React 就会在新的一次渲染后去对比前后两次的渲染，查看数组内是否有变量发生了更新（只要有一个数组元素变了，就会被认为更新发生了），并在有更新的前提下去触发 useEffect 中定义的副作用逻辑。

### Why React-Hooks：Hooks 是如何帮助我们升级工作模式的

在第 06 课时我们已经了解到，函数组件相比类组件来说，有着不少能够利好 React 组件开发的特性，而 React-Hooks 的出现正是为了强化函数组件的能力。现在，基于对 React-Hooks 编码层面的具体认知，想必你对“动机”的理解也已经上了一个台阶。这里我们就趁热打铁，针对“Why React-Hooks”这个问题，做一个加强版的总结。

相信有不少嗅觉敏锐的同学已经感觉到了——没错，这个环节就是手把手教你做“为什么需要 React-Hooks”这道面试题。以“Why xxx”开头的这种面试题，往往都没有标准答案，但会有一些关键的“点”，只要能答出关键的点，就足以证明你思考的方向是正确的，也就意味着这道题能给你加分。这里，我梳理了以下 4 条答题思路：

- 告别难以理解的 Class；

- 解决业务逻辑难以拆分的问题；

- 使状态逻辑复用变得简单可行；

- 函数组件从设计思想上来看，更加契合 React 的理念。

关于思路 4，我在上个课时已经讲得透透的了，这里我主要是借着代码的东风，把 1、2、3 摊开来给你看一下。

### 1. 告别难以理解的 Class：把握 Class 的两大“痛点”

坊间总有传言说 Class 是“难以理解”的，这个说法的背后是 this 和生命周期这两个痛点。

先来说说 this，在上个课时，你已经初步感受了一把 this 有多么难以捉摸。但那毕竟是个相对特殊的场景，更为我们所熟悉的，可能还是 React 自定义组件方法中的 this。看看下面这段代码：

```jsx
class Example extends Component {
  state = {
    name: '修言',
    age: '99';
  };
  changeAge() {
    // 这里会报错
    this.setState({
      age: '100'
    });
  }
  render() {
    return <button onClick={this.changeAge}>{this.state.name}的年龄是{this.state.age}</button>
  }
}
```

你先不用关心组件具体的逻辑，就看 changeAge 这个方法：它是 button 按钮的事件监听函数。当我点击 button 按钮时，希望它能够帮我修改状态，但事实是，点击发生后，程序会报错。原因很简单，changeAge 里并不能拿到组件实例的 this，至于为什么拿不到，我们将在第 15课时讲解其背后的原因，现在先不用关心。单就这个现象来说，略有一些 React 开发经验的同学应该都会非常熟悉。

> 在 React 中，**类组件**中的方法**默认不会绑定 this**，因此在调用类组件中的方法时，需要手动绑定 this 或使用箭头函数来绑定 this。

为了解决 this 不符合预期的问题，各路前端也是各显神通，之前用 bind、现在推崇箭头函数。但不管什么招数，本质上都是在用实践层面的约束来解决设计层面的问题。好在现在有了 Hooks，一切都不一样了，我们可以在函数组件里放飞自我（毕竟函数组件是不用关心 this 的）哈哈，解放啦！