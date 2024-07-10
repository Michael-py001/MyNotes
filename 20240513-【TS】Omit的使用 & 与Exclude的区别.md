# 20240513-【TS】Omit的使用 & 与Exclude的区别

在 TypeScript 中，`Omit<T, K>` 是一个实用的类型，它创建一个新的类型，这个新的类型是类型 `T`，但是去除了 `K` 中指定的属性。这在你想创建一个新的类型，这个新的类型和旧的类型很相似，但是去除了一些属性时非常有用。

`Omit` 类型需要两个参数：第一个参数是你想要修改的类型，第二个参数是一个字符串字面量的联合类型，表示你想要从第一个类型中去除的属性名。

## 去除单个属性

```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Omit<Todo, 'description'>;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};
```

![image-20240513161740339](https://s2.loli.net/2024/05/13/iAcjV3KegDUhmPT.png)

在这个例子中，`Todo` 类型有 `title`，`description` 和 `completed` 三个属性。`Omit<Todo, 'description'>` 创建了一个新的类型，这个新的类型和 `Todo` 类型一样，但是去除了 `description` 属性。所以，`TodoPreview` 类型只有 `title` 和 `completed` 两个属性。

你可以使用 `Omit` 类型来去除任何数量的属性，只需要将属性名作为 `Omit` 的第二个参数，用逗号分隔即可，如 `Omit<Todo, 'description' | 'completed'>`。

## 去除多个属性

如果你想从一个类型中去除多个属性，你可以在 `Omit` 的第二个参数中列出所有要去除的属性，用管道符 `|` 分隔。以下是一个例子：

```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
  createdAt: Date;
}

type TodoPreview = Omit<Todo, 'description' | 'createdAt'>;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};
```

在这个例子中，`Omit<Todo, 'description' | 'createdAt'>` 创建了一个新的类型，这个新的类型和 `Todo` 类型一样，但是去除了 `description` 和 `createdAt` 属性。所以，`TodoPreview` 类型只有 `title` 和 `completed` 两个属性。

![image-20240513161832483](https://s2.loli.net/2024/05/13/lIB2T5Wtr9eEPLM.png)

## Omit和Exclude有什么区别

`Omit` 和 `Exclude` 都是 TypeScript 中的实用类型，它们都用于创建新的类型，但是它们的用途和工作方式有所不同。

`Omit<T, K>` 创建一个新的类型，这个新的类型是类型 `T`，但是去除了 `K` 中指定的属性。例如：

```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Omit<Todo, 'description'>;
```

在这个例子中，`TodoPreview` 类型和 `Todo` 类型一样，但是去除了 `description` 属性。

而 `Exclude<T, U>` 创建一个新的类型，这个新的类型是类型 `T`，但是去除了与类型 `U` 可赋值的所有值。例如：

```ts
type T = number | string | boolean;
type U = Exclude<T, boolean | number>;
```

在这个例子中，`U` 类型是 `T` 类型去除了 `boolean` 和 `number` 后的结果，所以 `U` 类型只包含 `string`。

总的来说，`Omit` 用于**去除对象类型的某些属性**，而 `Exclude` 用于**从联合类型中去除某些类型**。