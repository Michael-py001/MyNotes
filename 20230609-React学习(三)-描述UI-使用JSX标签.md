# 20230609-React学习(三)-描述UI-使用JSX标签

**JSX** 是 JavaScript 语法扩展，可以让你在 JavaScript 文件中书写类似 HTML 的标签。虽然还有其它方式可以编写组件，但大部分 React 开发者更喜欢 JSX 的简洁性，并且在大部分代码库中使用它。

> ### 你将会学习到
>
> - 为什么 React 将标签和渲染逻辑耦合在一起
> - JSX 与 HTML 有什么区别
> - 如何通过 JSX 展示信息

## JSX: 将标签引入 JavaScript

网页是构建在 HTML、CSS 和 JavaScript 之上的。多年以来，web 开发者都是将网页内容存放在 HTML 中，样式放在 CSS 中，而逻辑则放在 JavaScript 中 —— 通常是在不同的文件中！页面的内容通过标签语言描述并存放在 HTML 文件中，而逻辑则单独存放在 JavaScript 文件中。

![image-20230609155246310](https://s2.loli.net/2023/06/09/imXvt9keKOZBUQa.png)

但随着 Web 的交互性越来越强，逻辑越来越决定页面中的内容。JavaScript 负责 HTML 的内容！这也是为什么 **在 React 中，渲染逻辑和标签共同存在于同一个地方——组件。**

![image-20230609155449064](https://s2.loli.net/2023/06/09/tOBnWMp1fY7sq2H.png)

将一个按钮的渲染逻辑和标签放在一起可以确保它们在每次编辑时都能保持互相同步。反之，彼此无关的细节是互相隔离的，例如按钮的标签和侧边栏的标签。这样我们在修改其中任意一个组件时会更安全。

每个 React 组件都是一个 JavaScript 函数，它会返回一些标签，React 会将这些标签渲染到浏览器上。React 组件使用一种被称为 JSX 的语法扩展来描述这些标签。JSX 看起来和 HTML 很像，但它的语法更加严格并且可以动态展示信息。了解这些区别最好的方式就是将一些 HTML 标签转化为 JSX 标签。

> - 注意
>
> [JSX and React 是相互独立的](https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html#whats-a-jsx-transform) 东西。但它们经常一起使用，但你 **可以** 单独使用它们中的任意一个，JSX 是一种语法扩展，而 React 则是一个 JavaScript 的库。

## 将 HTML 转化为 JSX

假设你现在有一些（完全有效的）HTML 标签，但是不能直接复制到组件中，需要转成jsx语法才行。

![image-20230609165217698](https://s2.loli.net/2023/06/09/lmKyxIB5EsL8nR9.png)

这是因为 JSX 语法更加严格并且相比 HTML 有更多的规则！上面的错误提示可以帮助你修复标签中的错误，当然也可以参考下面的指引。

> 大部分情况下，React 在屏幕上显示的错误提示就能帮你找到问题所在，如果在编写过程中遇到问题就参考一下提示吧。

## JSX 规则

### 1. 只能返回一个根元素

如果想要在一个组件中包含多个元素，**需要用一个父标签把它们包裹起来**。

例如，你可以使用一个 `<div>` 标签：

```html
<div>
  <h1>海蒂·拉玛的代办事项</h1>
  <img 
    src="https://i.imgur.com/yXOvdOSs.jpg" 
    alt="Hedy Lamarr" 
    class="photo"
  >
  <ul>
    ...
  </ul>
</div>
```

如果你不想在标签中增加一个额外的 `<div>`，可以用 `<>` 和 `</>` 元素来代替：

```diff
+ <>
  <h1>海蒂·拉玛的代办事项</h1>
  <img 
    src="https://i.imgur.com/yXOvdOSs.jpg" 
    alt="Hedy Lamarr" 
    class="photo"
  >
  <ul>
    ...
  </ul>
+ </>
```

这个空标签被称作 *[Fragment](https://zh-hans.react.dev/reference/react/Fragment)*. React Fragment 允许你将子元素分组，而不会在 HTML 结构中添加额外节点。

> ### 深入探讨:为什么多个 JSX 标签需要被一个父元素包裹？ 
>
> JSX 虽然看起来很像 HTML，但在底层其实被转化为了 JavaScript 对象，你不能在一个函数中返回多个对象，除非用一个数组把他们包装起来。这就是为什么多个 JSX 标签必须要用一个父元素或者 Fragment 来包裹。

### 2. 标签必须闭合

JSX 要求标签必须正确闭合。像 `<img>` 这样的自闭合标签必须书写成 `<img />`，而像 `<li>oranges` 这样只有开始标签的元素必须带有闭合标签，需要改为 `<li>oranges</li>`。

海蒂·拉玛的照片和代办事项的标签经修改后变为：

```tsx
<>
  <img 
    src="https://i.imgur.com/yXOvdOSs.jpg" 
    alt="Hedy Lamarr" 
    class="photo"
   />
  <ul>
      <li>发明一种新式交通信号灯</li>
      <li>排练一个电影场景</li>
      <li>改进频谱技术</li>
  </ul>
</>
```

### 3. 使用驼峰式命名法给大部分属性命名！

JSX 最终会被转化为 JavaScript，而 JSX 中的属性也会变成 JavaScript 对象中的键值对。在你自己的组件中，经常会遇到需要用变量的方式读取这些属性的时候。但 JavaScript 对变量的命名有限制。例如，变量名称不能包含 `-` 符号或者像 `class` 这样的保留字。

这就是为什么在 React 中，大部分 HTML 和 SVG 属性都用驼峰式命名法表示。例如，需要用 `strokeWidth` 代替 `stroke-width`。由于 `class` 是一个保留字，所以在 React 中需要用 `className` 来代替。这也是 [DOM 属性中的命名](https://developer.mozilla.org/en-US/docs/Web/API/Element/className):

```html
<img 
  src="https://i.imgur.com/yXOvdOSs.jpg" 
  alt="Hedy Lamarr" 
  className="photo"
/>
```

## 摘要

现在你知道了为什么我们需要 JSX 以及如何在组件中使用它：

- 由于渲染逻辑和标签是紧密相关的，所以 React 将它们存放在一个组件中。
- JSX 类似 HTML，不过有一些区别。如果需要的话可以使用 [转化器](https://transform.tools/html-to-jsx) 将 HTML 转化为 JSX。
- 错误提示通常会指引你将标签修改为正确的格式。