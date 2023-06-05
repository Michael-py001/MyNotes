# 20230601-TypeScript基础入门(八) - Mapped Types 映射类型 

当你不想重复自己时，有时一个类型需要基于另一个类型。

映射类型基于索引签名的语法构建，索引签名用于声明尚未提前声明的属性类型：

```ts
type OnlyBoolsAndHorses = {
  [key: string]: boolean | Horse;
};
 
const conforms: OnlyBoolsAndHorses = {
  del: true,
  rodney: false,
};
```

映射类型是一种泛型类型，它使用 `PropertyKey` s 的联合（通常通过 `keyof` 创建）来循环访问键以创建类型：

通常使用`in`遍历对象中的属性，而`keyof`通常用来取对象中的key值，

```ts
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};
```

在此示例中， `OptionsFlags` 将获取类型 `Type` 中的所有属性，并将其值更改为布尔值。

![image-20230601135026261](https://s2.loli.net/2023/06/01/wPl7AiRhNDaFQZ4.png)

## 映射修饰符

在映射过程中可以应用两个额外的修饰符： `readonly` 和 `?` ，它们分别影响可变性和可选性。\

您可以通过以 `-` 或 `+` 为前缀来删除或添加这些修饰符。如果不添加前缀，则假定为 `+` 。

![image-20230601135945619](https://s2.loli.net/2023/06/01/593HaxPeWpGTqFD.png)

![image-20230601140021027](https://s2.loli.net/2023/06/01/yUuFc3xdNfoHZVn.png)

```ts

type Concreate<Type> = {
  [Property in keyof Type]-? : Type[Property]; //去掉可选
}

type MaybeUser = {
  id: string;
  name?: string;
  age?: number;
};

type Iser = Concreate<MaybeUser>;
```

## Key Remapping via `as`    通过 `as` 进行键重新映射

在 TypeScript 4.1 及更高版本中，您可以使用映射类型中的 `as` 子句重新映射映射类型中的键：

```ts
type MappedTypeWithNewProperties<Type> = {
    [Properties in keyof Type as NewKeyType]: Type[Properties]
}
```

您可以利用模板文本类型等功能从以前的属性名称创建新的属性名称：

![image-20230601143437531](https://s2.loli.net/2023/06/01/PIk7dMiuograS9R.png)

可以通过条件类型生成 `never` 来筛选出键：

![image-20230601143608895](https://s2.loli.net/2023/06/01/4iQfAdJ2azwNTEO.png)

您可以映射任意联合，不仅仅是 `string | number | symbol` 的联合，而是任何类型的联合：

![image-20230601144355028](https://s2.loli.net/2023/06/01/DpBLd7VGAW1N9sI.png)

### Further Exploration 进一步探索

映射类型可与此类型操作部分中的其他功能很好地配合使用，例如，下面是使用条件类型的映射类型，该类型返回 `true` 或 `false` ，具体取决于对象是否将属性 `pii` 设置为文本 `true` ：

![image-20230601144712627](https://s2.loli.net/2023/06/01/PrmnzsW4wuKkNAt.png)