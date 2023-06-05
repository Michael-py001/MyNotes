# 20230531-TypeScript基础入门(六)-Indexed Access Types 索引访问类型 

> 摘自： [TypeScript: Documentation - 条件类型 (typescriptlang.org)](https://www.typescriptlang.org/zh/docs/handbook/2/conditional-types.html)

我们可以使用索引访问类型来查找另一个类型上的特定属性：

```ts
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"]; // type Age = number
```

索引类型本身就是一种类型，因此我们可以使用联合类型、keyof 或其他完全不同的类型:

![image-20230531162148013](https://s2.loli.net/2023/05/31/Fq6QBTpZzUjyXJx.png)

使用任意类型进行索引的另一个示例是使用 `number` 获取数组元素的类型。我们可以结合 `typeof` 来方便地捕获数组字面量的元素类型:

![image-20230531175944418](https://s2.loli.net/2023/05/31/EB6FwojAVZyN9Tk.png)

在进行索引时，您只能使用类型，这意味着您不能使用 `const` 来创建一个变量引用。

![image-20230531180052267](https://s2.loli.net/2023/05/31/pUOfAdQ7TVNJ8tG.png)

然而，您可以使用类型别名进行类似的重构：

```ts
type key = "age";
type Age = Person[key];
```

![image-20230601103125405](https://s2.loli.net/2023/06/01/cZWxoNHYfPb8pC9.png)

