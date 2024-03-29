# 20230619-React学习(十四)-添加交互-更新state中的对象

State 可以保存任何类型的 JavaScript 值，包括对象。但是你不能直接修改state中的对象。你要做的是，当你想更新一个对象的值时，你需要创建一个新对象(或者对你要修改的对象做一个拷贝值)，然后设置state为这个拷贝值。

> ### 你将会学习到
>
> - 如何正确更新state中的对象
> - 如何在不改变嵌套对象的情况下更新嵌套对象
> - 什么是不变性，以及如何不打破它
> - 如何使用 Immer 减少对象复制的重复性

## 什么是 mutation? 

You can store any kind of JavaScript value in state.
您可以在状态中存储任何类型的 JavaScript 值。

```ts
const [x, setX] = useState(0);
```

到目前为止，您一直在使用数字、字符串和布尔值。这些类型的JavaScript值是“不可变的”，这意味着不可更改或“只读”。您可以触发重新渲染以替换值：

```ts
setX(5);
```

state`x` 从 `0` 变为 `5` ，但数字 `0` 本身没有改变。无法对 JavaScript 中的内置基元值（如数字、字符串和布尔值）进行任何更改。

现在思考一个state中的对象：

```ts
const [position, setPosition] = useState({ x: 0, y: 0 });
```

从技术上讲，可以更改对象本身的内容。这称为突变：

```js
position.x = 5;
```

但是，尽管state中的对象在技术上是可变的，但您应该将它们视为不可变的，就像数字、布尔值和字符串一样。与其改变它们，不如始终替换它们。

## 将状态视为只读

换句话说，您应该将放入state的任何 JavaScript 对象视为只读。

本示例使一个对象处于状态以表示当前指针位置。当您在预览区域上触摸或移动光标时，红点应该会移动。但是点停留在初始位置：

<iframe src="https://codesandbox.io/embed/proud-dawn-gwbwpr?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="proud-dawn-gwbwpr"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

问题出在这段代码上。

```ts
onPointerMove={e => {
  position.x = e.clientX;
  position.y = e.clientY;
}}
```

此代码修改从[上一个渲染](https://zh-hans.react.dev/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time)中分配给 `position` 的对象。但是如果不使用`setPosition`设置函数，React 就不知道对象已经发生了变化。所以 React 没有做任何事情作为回应。这就像在你已经吃完饭后试图改变顺序。虽然在某些情况下可以更改状态，但我们不建议这样做。应将渲染中有权访问的状态值视为只读。

在这种情况下，要实际触发[重新渲染](https://zh-hans.react.dev/learn/state-as-a-snapshot#setting-state-triggers-renders)，请创建一个新对象并将其传递给状态设置函数：

```tsx
onPointerMove={e => {
  setPosition({
    x: e.clientX,
    y: e.clientY
  });
}}
```

用 `setPosition` ，你告诉 React：

- 将 `position` 替换为此新对象
- 并再次渲染此组件

请注意，当您触摸或悬停在预览区域上时，红点现在如何跟随您的指针：

<iframe src="https://codesandbox.io/embed/epic-kalam-zyozg3?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="epic-kalam-zyozg3"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

> #### Local mutation is fine
>
> 像这样的代码是一个问题，因为它修改了状态中的现有对象：
>
> ```js
> position.x = e.clientX;
> position.y = e.clientY;
> ```
>
> 但是像这样的代码绝对没问题，因为您正在更改刚刚创建的新对象：
>
> ```js
> const nextPosition = {};
> nextPosition.x = e.clientX;
> nextPosition.y = e.clientY;
> setPosition(nextPosition);
> ```
>
> 其实完全等同于写这个：
>
> ```js
> setPosition({
>   x: e.clientX,
>   y: e.clientY
> });
> ```
>
> 仅当您更改已处于`state`的现有对象时，突变才是一个问题。改变您刚刚创建的对象是可以的，因为还没有其他代码引用它。更改它不会意外影响依赖于它的东西。这被称为“局部突变”。您甚至可以在渲染时进行局部更改。非常方便，完全没问题！

## 使用展开语法复制对象

在前面的示例中， `position` 对象始终是从当前光标位置新鲜创建的。但通常，您需要将现有数据作为要创建的新对象的一部分包含在内。例如，您可能只想更新表单中的一个字段，但保留所有其他字段的先前值。

这些输入字段不起作用，因为 `onChange` 处理程序会改变状态：

<iframe src="https://codesandbox.io/embed/peaceful-thunder-65nfcr?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="peaceful-thunder-65nfcr"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

例如，这行代码从过去的渲染中改变state：

```js
person.firstName = e.target.value;
```

一个可靠的方法去实现上述需求就是创建一个新对象并将其传递给 `setPerson`，但在这里，您还希望将现有数据复制到其中，因为只有一个字段已更改：

```js
setPerson({
  firstName: e.target.value, // New first name from the input
  lastName: person.lastName,
  email: person.email
});
```

可以使用 `...` 对象展开语法，这样就不需要单独复制每个属性。

```js
setPerson({
  ...person, // Copy the old fields
  firstName: e.target.value // But override this one
});
```

现在表单运行正常了。

请注意，您没有为每个输入字段声明单独的状态变量。对于大型表单，将所有数据分组到一个对象中非常方便 — 只要正确更新即可！

<iframe src="https://codesandbox.io/embed/goofy-ellis-j99jb6?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="goofy-ellis-j99jb6"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

请注意， `...` 展开语法是“浅层”的，它只复制一层深的东西。这使得它很快，但这也意味着如果你想更新嵌套属性，你必须多次使用它。

## 深入探讨: 对多个字段使用单个事件处理程序

还可以使用对象定义中的 `[` 和 `]` 大括号来指定具有动态名称的属性。下面是相同的示例，但使用单个事件处理程序而不是三个不同的事件处理程序：

<iframe src="https://codesandbox.io/embed/wizardly-drake-g2lnrt?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="wizardly-drake-g2lnrt"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

`e.target.name` 获取到的是input标签上设置的name属性

## 更新嵌套对象

考虑一个嵌套的对象结构，如下所示：

```js
const [person, setPerson] = useState({
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
});
```

如果你想更新 `person.artwork.city` ，使用突变来做到这一点很容易：

```js
person.artwork.city = 'New Delhi';
```

但是在 React 中，您将状态视为不可变的！为了更改 `city` ，您首先需要生成新的 `artwork` 对象（预填充前一个对象中的数据），然后生成指向新 `artwork` 的新 `person` 对象：

```js
const nextArtwork = { ...person.artwork, city: 'New Delhi' };
const nextPerson = { ...person, artwork: nextArtwork };
setPerson(nextPerson);
```

或者，编写为单个函数调用：

```js
setPerson({
  ...person, // Copy other fields
  artwork: { // but replace the artwork
    ...person.artwork, // with the same one
    city: 'New Delhi' // but in New Delhi!
  }
});
```

这有点罗嗦，但在许多情况下都很好用：

<iframe src="https://codesandbox.io/embed/cool-jasper-geo3mo?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="cool-jasper-geo3mo"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

## 深入探讨：对象并非真正嵌套

像这样的对象在代码中显示为“嵌套”：

```js
let obj = {
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
};
```

但是，“嵌套”是思考对象行为的不准确方法。当代码执行时，没有“嵌套”对象这样的东西。你实际上是在看两个不同的对象：

```js
let obj1 = {
  title: 'Blue Nana',
  city: 'Hamburg',
  image: 'https://i.imgur.com/Sd1AgUOm.jpg',
};

let obj2 = {
  name: 'Niki de Saint Phalle',
  artwork: obj1
};
```

`obj1` 对象不在`obj2`里面。例如， `obj3` 也可以“指向” `obj1` ：

```js
let obj1 = {
  title: 'Blue Nana',
  city: 'Hamburg',
  image: 'https://i.imgur.com/Sd1AgUOm.jpg',
};

let obj2 = {
  name: 'Niki de Saint Phalle',
  artwork: obj1
};

let obj3 = {
  name: 'Copycat',
  artwork: obj1
};
```

如果要改变 `obj3.artwork.city` ，它将同时影响 `obj2.artwork.city` 和 `obj1.city` 。这是因为 `obj3.artwork` 、 `obj2.artwork` 和 `obj1` 是同一个对象。当您将对象视为“嵌套”时，很难看到这一点。相反，它们是单独的对象，通过属性相互“指向”。

## 使用 Immer 编写简洁的更新逻辑

> [immerjs/use-immer: Use immer to drive state with a React hooks (github.com)](https://github.com/immerjs/use-immer)

如果你的 state 有多层的嵌套，你或许应该考虑 [将其扁平化](https://zh-hans.react.dev/learn/choosing-the-state-structure#avoid-deeply-nested-state)。但是，如果你不想改变 state 的数据结构，你可能更喜欢用一种更便捷的方式来实现嵌套展开的效果。[Immer](https://github.com/immerjs/use-immer) 是一个非常流行的库，它可以让你使用简便但可以直接修改的语法编写代码，并会帮你处理好复制的过程。通过使用 Immer，你写出的代码看起来就像是你“打破了规则”而直接修改了对象：

```js
import { useImmer } from "use-immer";
const [person, updatePerson] = useImmer({
    name: "Michel",
    artwork: {
        city:'guangzhou'
    }
  });
updatePerson(draft => {
  draft.artwork.city = 'Lagos';
});
```

但与常规突变不同的是，它不会覆盖过去的状态！

### Immer是如何运行的?

由 Immer 提供的 `draft` 是一种特殊类型的对象，被称为 [Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)，它会记录你用它所进行的操作。这就是你能够随心所欲地直接修改对象的原因所在！从原理上说，Immer 会弄清楚 `draft` 对象的哪些部分被改变了，并会依照你的修改创建出一个全新的对象。

## 深入探讨：为什么在 React 中不推荐直接修改 state？

有以下几个原因：

- **调试**：如果你使用 `console.log` 并且不直接修改 state，你之前日志中的 state 的值就不会被新的 state 变化所影响。这样你就可以清楚地看到两次渲染之间 state 的值发生了什么变化
- **优化**：React 常见的 [优化策略](https://zh-hans.react.dev/reference/react/memo) 依赖于如果之前的 props 或者 state 的值和下一次相同就跳过渲染。如果你从未直接修改 state ，那么你就可以很快看到 state 是否发生了变化。如果 `prevObj === obj`，那么你就可以肯定这个对象内部并没有发生改变。
- **新功能**：我们正在构建的 React 的新功能依赖于 state 被 [像快照一样看待](https://zh-hans.react.dev/learn/state-as-a-snapshot) 的理念。如果你直接修改 state 的历史版本，可能会影响你使用这些新功能。
- **需求变更**：有些应用功能在不出现任何修改的情况下会更容易实现，比如实现撤销/恢复、展示修改历史，或是允许用户把表单重置成某个之前的值。这是因为你可以把 state 之前的拷贝保存到内存中，并适时对其进行再次使用。如果一开始就用了直接修改 state 的方式，那么后面要实现这样的功能就会变得非常困难。
- **更简单的实现**：React 并不依赖于 mutation ，所以你不需要对对象进行任何特殊操作。它不需要像很多“响应式”的解决方案一样去劫持对象的属性、总是用代理把对象包裹起来，或者在初始化时做其他工作。这也是为什么 React 允许你把任何对象存放在 state 中——不管对象有多大——而不会造成有任何额外的性能或正确性问题的原因。

在实践中，你经常可以“侥幸”直接修改 state 而不出现什么问题，但是我们强烈建议你不要这样做，这样你就可以使用我们秉承着这种理念开发的 React 新功能。未来的贡献者甚至是你未来的自己都会感谢你的！

## 摘要

- 将 React 中所有的 state 都视为不可直接修改的。

- 当你在 state 中存放对象时，直接修改对象并不会触发重渲染，并会改变前一次渲染“快照”中 state 的值。
- 不要直接修改一个对象，而要为它创建一个 **新** 版本，并通过把 state 设置成这个新版本来触发重新渲染。
- 你可以使用这样的 `{...obj, something: 'newValue'}` 对象展开语法来创建对象的拷贝。
- 对象的展开语法是浅层的：它的复制深度只有一层。
- 想要更新嵌套对象，你需要从你更新的位置开始自底向上为每一层都创建新的拷贝。
- 想要减少重复的拷贝代码，可以使用 Immer。