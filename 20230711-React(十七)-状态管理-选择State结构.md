# 20230711-React(十七)-状态管理-选择State结构

构建良好的 state 可以让组件变得易于修改和调试，而不会经常出错。以下是你在构建 state 时应该考虑的一些建议。

> ### 你将会学习到
>
> - 使用单个 state 变量还是多个 state 变量
> - 组织 state 时应避免的内容
> - 如何解决 state 结构中的常见问题

## 构建 state 的原则 

当你编写一个存有 state 的组件时，你需要选择使用多少个 state 变量以及它们都是怎样的数据格式。尽管选择次优的 state 结构下也可以编写正确的程序，但有几个原则可以指导您做出更好的决策：

1. **合并关联的 state**。如果你总是同时更新两个或更多的 state 变量，请考虑将它们合并为一个单独的 state 变量。
2. **避免互相矛盾的 state**。当 state 结构中存在多个相互矛盾或“不一致”的 state 时，你就可能为此会留下隐患。应尽量避免这种情况。
3. **避免冗余的 state**。如果你能在渲染期间从组件的 props 或其现有的 state 变量中计算出一些信息，则不应将这些信息放入该组件的 state 中。
4. **避免重复的 state**。当同一数据在多个 state 变量之间或在多个嵌套对象中重复时，这会很难保持它们同步。应尽可能减少重复。
5. **避免深度嵌套的 state**。深度分层的 state 更新起来不是很方便。如果可能的话，最好以扁平化方式构建 state。

这些原则背后的目标是 **使 state 易于更新而不引入错误**。从 state 中删除冗余和重复数据有助于确保所有部分保持同步。这类似于数据库工程师想要 [“规范化”数据库结构](https://docs.microsoft.com/zh-CN/office/troubleshoot/access/database-normalization-description)，以减少出现错误的机会。用爱因斯坦的话说，**“让你的状态尽可能简单，但不要过于简单。”**

现在让我们来看看这些原则在实际中是如何应用的。

## 合并关联的 state

有时候你可能会不确定是使用单个 state 变量还是多个 state 变量。

你会像下面这样做吗？

```js
const [x, setX] = useState(0);
const [y, setY] = useState(0);
```

或这样？

```js
const [position, setPosition] = useState({ x: 0, y: 0 });
```

从技术上讲，你可以使用其中任何一种方法。但是，**如果某两个 state 变量总是一起变化，则将它们统一成一个 state 变量可能更好**。这样你就不会忘记让它们始终保持同步，就像下面这个例子中，移动光标会同时更新红点的两个坐标：

<iframe src="https://codesandbox.io/embed/mutable-moon-k3wul4?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="mutable-moon-k3wul4"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

另一种情况是，你将数据整合到一个对象或一个数组中时，你不知道需要多少个 state 片段。例如，当你有一个用户可以添加自定义字段的表单时，这将会很有帮助。

> ### 陷阱
>
> 如果你的 state 变量是一个对象时，请记住，[你不能只更新其中的一个字段](https://zh-hans.react.dev/learn/updating-objects-in-state) 而不显式复制其他字段。例如，在上面的例子中，你不能写成 `setPosition({ x: 100 })`，因为它根本就没有 `y` 属性! 相反，如果你想要仅设置 `x`，则可执行 `setPosition({ ...position, x: 100 })`，或将它们分成两个 state 变量，并执行 `setX(100)`。

## 避免矛盾的 state 

下面是带有 `isSending` 和 `isSent` 两个 state 变量的酒店反馈表单：

<iframe src="https://codesandbox.io/embed/pedantic-mclean-km5s56?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="pedantic-mclean-km5s56"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

尽管这段代码是有效的，但也会让一些 state “极难处理”。例如，如果你忘记同时调用 `setIsSent` 和 `setIsSending`，则可能会出现 `isSending` 和 `isSent` 同时为 `true` 的情况。你的组件越复杂，你就越难理解发生了什么。

**因为 `isSending` 和 `isSent` 不应同时为 `true`，所以最好用一个 `status` 变量来代替它们，这个 state 变量可以采取三种有效状态其中之一**：`'typing'` (初始), `'sending'`, 和 `'sent'`:

<iframe src="https://codesandbox.io/embed/adoring-sound-4nc85z?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="adoring-sound-4nc85z"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

你仍然可以声明一些常量，以提高可读性：

```js
const isSending = status === 'sending';
const isSent = status === 'sent';
```

但它们不是 state 变量，所以你不必担心它们彼此失去同步。

## 避免冗余的 state 

如果你能在渲染期间从组件的 props 或其现有的 state 变量中计算出一些信息，则不应该把这些信息放到该组件的 state 中。

例如，以这个表单为例。它可以运行，但你能找到其中任何冗余的 state 吗？

<iframe src="https://codesandbox.io/embed/musing-silence-dnvs2u?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="musing-silence-dnvs2u"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

这个表单有三个 state 变量：`firstName`、`lastName` 和 `fullName`。然而，`fullName` 是多余的。**在渲染期间，你始终可以从 `firstName` 和 `lastName` 中计算出 `fullName`，因此需要把它从 state 中删除。**

你可以这样做：

<iframe src="https://codesandbox.io/embed/amazing-lena-sc71ts?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="amazing-lena-sc71ts"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

这里的 `fullName` **不是** 一个 state 变量。相反，它是在渲染期间中计算出的：

```js
const fullName = firstName + ' ' + lastName;
```

因此，更改处理程序不需要做任何特殊操作来更新它。当你调用 `setFirstName` 或 `setLastName` 时，你会触发一次重新渲染，然后下一个 `fullName` 将从新数据中计算出来。

### 深入探讨: 不要在 state 中镜像 props 

以下代码是体现 state 冗余的一个常见例子：

```js
function Message({ messageColor }) {
  const [color, setColor] = useState(messageColor);
}
```

这里，一个 `color` state 变量被初始化为 `messageColor` 的 prop 值。这段代码的问题在于，**如果父组件稍后传递不同的 `messageColor` 值（例如，将其从 `'blue'` 更改为 `'red'`），则 `color`** state 变量**将不会更新！** state 仅在第一次渲染期间初始化。

这就是为什么在 state 变量中，“镜像”一些 prop 属性会导致混淆的原因。相反，你要在代码中直接使用 `messageColor` 属性。如果你想给它起一个更短的名称，请使用常量：

```js
ction Message({ messageColor }) {
  const color = messageColor
 }
```

这种写法就不会与从父组件传递的属性失去同步。

只有当你 **想要** 忽略特定 props 属性的所有更新时，将 props “镜像”到 state 才有意义。按照惯例，prop 名称以 `initial` 或 `default` 开头，以阐明该 prop 的新值将被忽略：

```js
function Message({ initialColor }) {
  // 这个 `color` state 变量用于保存 `initialColor` 的 **初始值**。
  // 对于 `initialColor` 属性的进一步更改将被忽略。
  const [color, setColor] = useState(initialColor);
```

## 避免重复的 state 

下面这个菜单列表组件可以让你在多种旅行小吃中选择一个：

<iframe src="https://codesandbox.io/embed/nice-mclean-0m3ydb?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="nice-mclean-0m3ydb"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

当前，它将所选元素作为对象存储在 `selectedItem` state 变量中。然而，这并不好：**`selectedItem` 的内容与 `items` 列表中的某个项是同一个对象。** 这意味着关于该项本身的信息在两个地方产生了重复。

为什么这是个问题？让我们使每个项目都可以编辑：

<iframe src="https://codesandbox.io/embed/boring-wu-769ixj?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="boring-wu-769ixj"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

请注意，如果你首先单击菜单上的“Choose” **然后** 编辑它，**输入会更新，但底部的标签不会反映编辑内容。** 这是因为你有重复的 state，并且你忘记更新了 `selectedItem`。

尽管你也可以更新 `selectedItem`，但更简单的解决方法是消除重复项。在下面这个例子中，你将 `selectedId` 保存在 state 中，而不是在 `selectedItem` 对象中（它创建了一个与 `items` 内重复的对象），**然后** 通过搜索 `items` 数组中具有该 ID 的项，以此获取 `selectedItem`：

<iframe src="https://codesandbox.io/embed/tender-frog-ubzyo9?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="tender-frog-ubzyo9"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

(或者，你可以将所选索引保持在 state 中。)

state 过去常常是这样复制的：

- `items = [{ id: 0, title: 'pretzels'}, ...]`
- `selectedItem = {id: 0, title: 'pretzels'}`

改了之后是这样的：

- `items = [{ id: 0, title: 'pretzels'}, ...]`
- `selectedId = 0`

重复的 state 没有了，你只保留了必要的 state！

state 过去常常是这样复制的：

- `items = [{ id: 0, title: 'pretzels'}, ...]`
- `selectedItem = {id: 0, title: 'pretzels'}`

改了之后是这样的：

- `items = [{ id: 0, title: 'pretzels'}, ...]`
- `selectedId = 0`

重复的 state 没有了，你只保留了必要的 state！

现在，如果你编辑 **selected** 元素，下面的消息将立即更新。这是因为 `setItems` 会触发重新渲染，而 `items.find(...)` 会找到带有更新文本的元素。你不需要在 state 中保存 **选定的元素**，因为只有 **选定的 ID** 是必要的。其余的可以在渲染期间计算。

## 避免深度嵌套的 state 

想象一下，一个由行星、大陆和国家组成的旅行计划。你可能会尝试使用嵌套对象和数组来构建它的 state，就像下面这个例子：

<iframe src="https://codesandbox.io/embed/aged-lake-5m56js?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="aged-lake-5m56js"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

现在，假设你想添加一个按钮来删除一个你已经去过的地方。你会怎么做呢？[更新嵌套的 state](https://zh-hans.react.dev/learn/updating-objects-in-state#updating-a-nested-object) 需要从更改部分一直向上复制对象。删除一个深度嵌套的地点将涉及复制其整个父级地点链。这样的代码可能非常冗长。

**如果 state 嵌套太深，难以轻松更新，可以考虑将其“扁平化”。** 这里有一个方法可以重构上面这个数据。不同于树状结构，每个节点的 `place` 都是一个包含 **其子节点** 的数组，你可以让每个节点的 `place` 作为数组保存 **其子节点的 ID**。然后存储一个节点 ID 与相应节点的映射关系。

这个数据重组可能会让你想起看到一个数据库表：

<iframe src="https://codesandbox.io/embed/pensive-bessie-euroqm?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="pensive-bessie-euroqm"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

**现在 state 已经“扁平化”（也称为“规范化”），更新嵌套项会变得更加容易。**

现在要删除一个地点，您只需要更新两个 state 级别：

- 其 **父级** 地点的更新版本应该从其 `childIds` 数组中排除已删除的 ID。
- 其根级“表”对象的更新版本应包括父级地点的更新版本。

下面是展示如何处理它的一个示例：

<iframe src="https://codesandbox.io/embed/wizardly-glade-hfs4oo?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="wizardly-glade-hfs4oo"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

你确实可以随心所欲地嵌套 state，但是将其“扁平化”可以解决许多问题。这使得 state 更容易更新，并且有助于确保在嵌套对象的不同部分中没有重复。

### 深入探讨: 改善内存使用

理想情况下，您还应该从“表”对象中删除已删除的项目（以及它们的子项！）以改善内存使用。还可以 [使用 Immer](https://zh-hans.react.dev/learn/updating-objects-in-state#write-concise-update-logic-with-immer) 使更新逻辑更加简洁。

<iframe src="https://codesandbox.io/embed/nostalgic-khayyam-d01jrx?fontsize=14&hidenavigation=1&module=%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="nostalgic-khayyam-d01jrx"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

有时候，你也可以通过将一些嵌套 state 移动到子组件中来减少 state 的嵌套。这对于不需要保存的短暂 UI 状态非常有效，比如一个选项是否被悬停。

## 摘要

- 如果两个 state 变量总是一起更新，请考虑将它们合并为一个。
- 仔细选择你的 state 变量，以避免创建“极难处理”的 state。
- 用一种减少出错更新的机会的方式来构建你的 state。
- 避免冗余和重复的 state，这样您就不需要保持同步。
- 除非您特别想防止更新，否则不要将 props **放入** state 中。
- 对于选择类型的 UI 模式，请在 state 中保存 ID 或索引而不是对象本身。
- 如果深度嵌套 state 更新很复杂，请尝试将其展开扁平化。

## 全选操作

- 使用普通数组

  ```js
  const [selectedIds, setSelectedIds] = useState([]);
  
    const selectedCount = selectedIds.length;
  
    function handleToggle(toggledId) {
      // 它以前是被选中的吗？
      if (selectedIds.includes(toggledId)) {
        // Then remove this ID from the array.
        setSelectedIds(selectedIds.filter(id =>
          id !== toggledId
        ));
      } else {
        // 否则，增加 ID 到数组中。
        setSelectedIds([
          ...selectedIds,
          toggledId
        ]);
      }
    }
  ```

  

使用数组的一个小缺点是，对于每个项目，你都需要调用 `selectedIds.includes(letter.id)` 来检查它是否被选中。如果数组非常大，则这可能会成为性能问题，因为带有 [`includes()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/includes) 的数组搜索需要线性时间，并且你正在为每个单独的项目执行此搜索。

要解决这个问题，你可以在 state 中使用一个 [Set](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set) 对象，它提供了快速的 [`has()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set/has) 操作：

- 使用`Set()`判断是否存在

  ```js
  const [selectedIds, setSelectedIds] = useState(
      new Set()
    );
  
    const selectedCount = selectedIds.size;
  
    function handleToggle(toggledId) {
      // Create a copy (to avoid mutation).
      const nextIds = new Set(selectedIds);
      if (nextIds.has(toggledId)) {
        nextIds.delete(toggledId);
      } else {
        nextIds.add(toggledId);
      }
      setSelectedIds(nextIds);
    }
  ```



