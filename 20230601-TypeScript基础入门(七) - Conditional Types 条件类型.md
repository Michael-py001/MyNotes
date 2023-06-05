# 20230531-TypeScript基础入门(七) - Conditional Types 条件类型

大多数有效程序的核心是，我们必须依据输入做出一些决定。 JavaScript 程序也是如此，但是由于值可以很容易地被内省，这些决定也是基于输入的类型。 *条件类型* 有助于描述输入和输出类型之间的关系。

![image-20230601094938260](https://s2.loli.net/2023/06/01/EvgaWbfS6DznBVu.png)

条件类型看起来有点像 JavaScript 中的条件表达式（`条件 ? true 表达式 : false 表达式`）：

```ts
  SomeType extends OtherType ? TrueType : FalseType;
```

从上面的例子中，条件类型可能不会立即显得很有用，但是条件类型的威力来自于将它们与泛型一起使用。

让我们以下面的 `createLabel` 函数为例:

```ts
interface IdLabel {
  id: number /* 一些字段 */;
}
interface NameLabel {
  name: string /* 其它字段 */;
}
 
function createLabel(id: number): IdLabel;
function createLabel(name: string): NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel {
  throw "unimplemented";
}
```

这些 createLabel 的重载描述了单个基于输入类型进行选择的 JavaScript 函数。注意以下几点：

1. 如果一个库不得不在其 API 中一遍又一遍地做出相同的选择，这就变得很麻烦。
2. 我们必须创建三个重载：一种用于我们 *确定* 类型时的每种情况（一个用于 `string`，一个用于 `number`），一个用于最一般的情况（接受一个 `string | number`）。对于 `createLabel` 可以处理的每个新类型，重载的数量都会呈指数增长。

相反，我们可以将该逻辑转换为条件类型：

```ts
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;
```

然后，我们可以使用该条件类型将重载简化为没有重载的单个函数。

![image-20230601113916935](https://s2.loli.net/2023/06/01/gJM73lqkCITHXmp.png)

### 条件类型约束

通常，条件类型的检查将为我们提供一些新信息。 就像使用类型守卫缩小范围可以给我们提供更具体的类型一样，条件类型的 true 分支将根据我们检查的类型进一步约束泛型。

让我们来看看下面的例子：

![image-20230601103451909](https://s2.loli.net/2023/06/01/18IkvrNzMd2UCya.png)

在本例中，TypeScript 产生错误是因为不知道 `T` 有一个名为 `message` 的属性。 我们可以约束 `T`，TypeScript 也不会再抱怨了：

![image-20230601103608154](https://s2.loli.net/2023/06/01/pWXPaDV25r1dSE6.png)

然而，如果我们希望 `MessageOf` 接受任何类型，并且在 `message` 属性不可用的情况下默认为 `never` 之类的类型，我们应该怎么做呢？ 我们可以通过移出约束并引入条件类型来实现这一点：

![image-20230601103814266](https://s2.loli.net/2023/06/01/qblpuzhwD9IQocd.png)

在 true 分支中，TypeScript 知道 `T` *将* 有一个 `message` 属性。

作为另一个示例，我们还可以编写一个名为 `Flatten` 的类型，它将数组类型扁平为它们的元素类型，但在其他情况下不会处理它们：

![image-20230601104012775](https://s2.loli.net/2023/06/01/8AlbaDtUqCXruOP.png)

当 `Flatten` 被赋予数组类型时，它使用带 `number` 的索引访问来提取 `string[]` 的元素类型。 否则，它只返回给定的类型。

### 在条件类型中推断   `infer`关键字

我们发现自己使用条件类型来应用约束，然后提取出类型。 这最终成为一种非常常见的操作，条件类型使其变得更容易。

条件类型为我们提供了一种使用 `infer` 关键字从 true 分支中与之进行比较的类型中进行推断的方法。 例如，我们可以在 `Flatten` 中推断元素类型，而不是使用索引访问类型“手动”提取它：

```ts
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;
```

在这里，我们使用 `infer` 关键字以声明方式引入一个名为 `Item` 的新泛型类型变量，而不是指定如何在 true 分支中检索元素类型 `T`。 这使我们不必考虑如何挖掘和探索我们感兴趣的类型的结构。

### `infer`关键字的使用



我们可以使用 `infer` 关键字编写一些有用的助手类型别名。`infer` 是 TypeScript 中的一个关键字，用于在条件类型中推断类型变量。具体来说，`infer` 可以用于从一个类型中提取出另一个类型，并将其赋值给一个类型变量。

下面是一个使用 `infer` 的例子，假设我们有一个类型 `Person`，它包含一个名为 `name` 的属性和一个名为 `age` 的属性：

```ts
type Person = {
  name: string;
  age: number;
};
```

现在我们想从 `Person` 类型中提取出 `name` 属性的类型，可以使用 `infer` 关键字：

```ts
type GetName<T> = T extends { name: infer U; } ? U : never;

type Name = GetName<Person>; // string
```

在上面的例子中，我们定义了一个名为 `GetName` 的条件类型，它接受一个类型参数 `T`。如果 `T` 是一个包含 `name` 属性的对象类型，那么我们使用 `infer` 关键字从 `name` 属性中提取出它的类型，并将其赋值给类型变量 `U`。最后，我们将 `U` 作为 `GetName` 的返回类型。

通过这种方式，我们可以从一个类型中提取出另一个类型，并将其用于其他类型的定义中。



 例如，对于简单的情况，我们可以从函数类型中提取返回类型：

![image-20230601110233677](https://s2.loli.net/2023/06/01/wNd6YzO29AGS3jL.png)

当从具有多个调用签名的类型（如重载函数的类型）进行推断时，将从 *最后一个* 签名进行推断（这也许是最宽松的万能情况）。无法基于参数类型列表执行重载决议。

![image-20230601111446230](https://s2.loli.net/2023/06/01/IEr1aqySgROHxp2.png)

## 分配条件类型

当传入的类型参数为联合类型时，他们会被 *分配类型* 。 以下面的例子为例：

```ts
type ToArray<Type> = Type extends any ? Type[] : never;
```

如果我们将联合类型传入 `ToArray`，则条件类型将应用于该联合类型的每个成员。

![image-20230601111904341](https://s2.loli.net/2023/06/01/fQESHFGn5r9XPVj.png)

这里发生的情况是 `StrOrNumArray` 分布在以下位置：

```ts
 string | number;
```

并在联合类型的每个成员类型上映射到有效的内容：

```ts
ToArray<string> | ToArray<number>;
```

所以我们得到：

```ts
string[] | number[];
```

通常，分布性是所需的行为。 要避免这种行为，可以用方括号括起 `extends` 关键字的两边。

![image-20230601112109971](https://s2.loli.net/2023/06/01/lJLcb7hKpmE3ta9.png)



> 详解

当你需要根据某个类型的属性来确定另一个类型的时候，可以使用 TypeScript 中的条件类型。其中，分配条件类型是一种常见的条件类型，它可以根据某个类型的属性来确定另一个类型的属性。

下面是一个使用分配条件类型的例子，假设我们有一个名为 `Person` 的类型，它包含一个名为 `name` 的属性和一个名为 `age` 的属性：

```ts
type Person = {
  name: string;
  age: number;
};
```

现在我们想定义一个名为 `PartialBy` 的类型，它接受两个类型参数 `T` 和 `K`，其中 `T` 是一个对象类型，`K` 是 `T` 中的一个属性名。`PartialBy` 的作用是将 `T` 中的 `K` 属性变为可选属性。

可以使用分配条件类型来实现 `PartialBy`：

```ts
type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

type PersonPartialByAge = PartialBy<Person, 'age'>;
// { name: string; age?: number; }
```

在上面的例子中，我们定义了一个名为 `PartialBy` 的类型，它接受两个类型参数 `T` 和 `K`。首先，我们使用 `Omit<T, K>` 来从 `T` 中排除 `K` 属性，然后使用 `Partial<Pick<T, K>>` 来将 `K` 属性变为可选属性。最后，我们将这两个类型合并起来，得到一个新的类型。

通过这种方式，我们可以根据某个类型的属性来确定另一个类型的属性，从而实现更加灵活的类型定义。

### Omit是什么？

`Omit` 是 TypeScript 中的一个工具类型，用于从一个类型中排除指定的属性。具体来说，`Omit<T, K>` 可以创建一个新的类型，它包含了 `T` 类型中除了 `K` 属性之外的所有属性。

下面是一个使用 `Omit` 的例子，假设我们有一个名为 `Person` 的类型，它包含一个名为 `name` 的属性和一个名为 `age` 的属性：

```ts
type Person = {
  name: string;
  age: number;
};
```

现在我们想创建一个新的类型，它排除了 `Person` 类型中的 `age` 属性，可以使用 `Omit`：

```ts
type PersonWithoutAge = Omit<Person, 'age'>;
// { name: string; }
```

在上面的例子中，我们使用 `Omit<Person, 'age'>` 创建了一个新的类型 `PersonWithoutAge`，它包含了 `Person` 类型中除了 `age` 属性之外的所有属性。这样就可以方便地从一个类型中排除指定的属性了。
