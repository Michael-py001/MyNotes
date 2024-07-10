# 使用web components 实现todo list

> 翻译自：
>
> https://jakelazaroff.com/words/web-components-eliminate-javascript-framework-lock-in/

## 什么是Web Component?

如果你不熟悉Web组件，这里有一个关于它们如何工作的简单入门介绍。

首先，我们在JavaScript中声明 `HTMLElement` 的一个子类。我们称其为 `MyComponent`

```js
class MyComponent extends HTMLElement {
  constructor() {
    super();
    this.shadow = this.attachShadow({ mode: "open" });
  }

  connectedCallback() {
    this.shadow.innerHTML = `
      <p>Hello from a web component!</p>
      <style>
        p {
          color: pink;
          font-weight: bold;
          padding: 1rem;
          border: 4px solid pink;
        }
      </style>

    `;
  }
}
```

构造函数中对 `attachShadow` 的调用使我们的组件使用影子DOM，它将来自页面其余部分的标记和样式封装在我们的组件中。 `connectedCallback` 在Web组件实际连接到DOM树时调用，将HTML内容呈现到组件的“影子根”中。

这预示着我们将如何让我们的框架与Web组件一起工作。 [1](https://jakelazaroff.com/words/web-components-eliminate-javascript-framework-lock-in/#user-content-fn-rimshot) 我们通常将框架“附加”到一个DOM元素，然后让该框架接管该元素的所有后代。使用Web组件，我们可以将框架附加到影子根，这确保它只能访问组件的“影子树”。

接下来，我们为 `MyComponent` 类定义一个定制元素名称：

```js
customElements.define("my-component", MyComponent);
```

每当具有该自定义元素名称的标记出现在页面上时，对应的DOM节点实际上是 `MyComponent` 的实例！

```html
<my-component></my-component>
<script>
  const myComponent = document.querySelector("my-component");
  console.log(myComponent instanceof MyComponent); // true
</script>
```

## 使用React开发TodoApp

```jsx
// TodoApp.jsx
export default function TodoApp() {
  return <></>;
}
```

我们可以在这里开始添加元素来屏蔽基本的DOM结构，但我想为此编写另一个组件，以展示我们如何以嵌套框架组件的相同方式来嵌套Web组件。

大多数框架都像普通的HTML元素一样，通过嵌套来支持组合。从外面看，它通常是这样的：

```html
<Card>
  <Avatar />
</Card>
```

在内部，框架有几种方法来处理这一问题。例如，React和Solid使您可以将这些子项作为特殊 `children` props进行访问：

```jsx
function Card(props) {
  return <div class="card">{props.children}</div>;
}
```

对于使用阴影DOM的Web组件，我们可以使用 `<slot>` 元素来做同样的事情。当浏览器遇到 `<slot>` 时，它会将其替换为Web组件的子项。

`<slot>` 实际上比React或Solid的 `children` 更强大。如果我们给每个槽一个 `name` 属性，一个Web组件可以有多个 `<slot>` S，我们可以通过赋予它一个与 `<slot>` ‘S `name` 匹配的 `slot` 属性来确定每个嵌套元素的去向。

让我们看看这在实践中是什么样子的。我们将使用Solid编写布局组件：

```jsx
// TodoLayout.jsx
import { render } from "solid-js/web";

function TodoLayout() {
  return (
    <div class="wrapper">
      <header class="header">
        <slot name="title" />
        <slot name="filters" />
      </header>
      <div>
        <slot name="todos" />
      </div>
      <footer>
        <slot name="input" />
      </footer>
    </div>
  );
}

customElements.define(
  "todo-layout",
  class extends HTMLElement {
    constructor() {
      super();
      this.shadow = this.attachShadow({ mode: "open" });
    }

    connectedCallback() {
      render(() => <TodoLayout />, this.shadow);
    }
  }
);
```

我们的Solid  Web组件有两个部分：顶部的Web组件包装和底部的实际实体组件。

关于Solid组件，最需要注意的是，我们使用的是名为 `<slot>` S而不是 `children` props。 `children` 是Solid处理的，只会让我们嵌套其他Solid组件， `<slot>` S是由浏览器自己处理的，可以让我们嵌套任何Html元素--包括用其他框架编写的Web组件！

Web组件包装器与上面的示例非常类似。它在构造函数中创建一个影子根，然后在 `connectedCallback` 方法中呈现实体组件。

请注意，这不是Web组件包装器的完整实现！至少，我们可能希望定义一个 `attributeChangedCallback` 方法，以便在属性更改时可以重新呈现实体组件。如果您在生产中使用它，您可能应该使用Solid提供的名为Solid Element的包来为您处理所有这些问题。

回到我们的React应用程序中，我们现在可以使用我们的组件：

```jsx
// TodoApp.jsx
export default function TodoApp() {
  return (
    <todo-layout>
      <h1 slot="title">Todos</h1>
    </todo-layout>
  );
}
```

效果如下：

![image-20240704143309729](https://s2.loli.net/2024/07/04/LgmwSz8H93MrD5d.png)

## 使用nanostores创建一个框架通用的状态管理存储

```js
// store.js
import { atom, computed } from "nanostores";

let id = 0;
export const $todos = atom([]);
export const $done = computed($todos, todos => todos.filter(todo => todo.done));
export const $left = computed($todos, todos => todos.filter(todo => !todo.done));

export function addTodo(text) {
  $todos.set([...$todos.get(), { id: id++, text }]);
}

export function checkTodo(id, done) {
  $todos.set($todos.get().map(todo => (todo.id === id ? { ...todo, done } : todo)));
}

export function removeTodo(id) {
  $todos.set($todos.get().filter(todo => todo.id !== id));
}

export const $filter = atom("all");
```

使用vue编写

```vue
<!-- TodoFilters.ce.vue -->
<script setup>
  import { useStore, useVModel } from "@nanostores/vue";

  import { $todos, $done, $left, $filter } from "./store.js";

  const filter = useVModel($filter);
  const todos = useStore($todos);
  const done = useStore($done);
  const left = useStore($left);
</script>

<template>
  <div>
    <label>
      <input type="radio" name="filter" value="all" v-model="filter" />
      <span> All ({{ todos.length }})</span>
    </label>
    <label>
      <input type="radio" name="filter" value="todo" v-model="filter" />
      <span> Todo ({{ left.length }})</span>
    </label>

    <label>
      <input type="radio" name="filter" value="done" v-model="filter" />
      <span> Done ({{ done.length }})</span>
    </label>
  </div>
</template>
```

Vue并不像Svelte那样将组件编译为自定义元素那么简单。我们需要创建另一个文件，导入Vue组件并调用它：

```js
// TodoFilters.js
import { defineCustomElement } from "vue";

import TodoFilters from "./TodoFilters.ce.vue";

customElements.define("todo-filters", defineCustomElement(TodoFilters));
```

