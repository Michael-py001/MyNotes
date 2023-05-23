# 20230517-TypeScript基础入门(二) - Object Types

在 JavaScript 中，我们分组和传递数据的基本方式是通过对象。在 TypeScript 中，我们通过对象类型来表示它们。

正如我们所见，它们可以是匿名的：

```ts
function greet(person: { name: string; age: number }) {
  return "Hello " + person.name;
}
```

或者它们可以通过使用任一接口来命名

```ts
interface Person {
  name: string;
  age: number;
}
 
function greet(person: Person) {
  return "Hello " + person.name;
}
```

或类型别名

```ts
type Person = {
  name: string;
  age: number;
};
 
function greet(person: Person) {
  return "Hello " + person.name;
}
```

# Property Modifiers 属性修饰符

对象类型中的每个属性都可以指定几件事：类型、属性是否可选以及属性是否可以写入。

## Optional Properties 可选属性

很多时候，我们会发现自己在处理可能具有属性集的对象。在这些情况下，我们可以通过在名称末尾添加问号 ( `?` ) 将这些属性标记为可选。

```ts
interface PaintOptions {
  shape: Shape;
  xPos?: number;
  yPos?: number;
}
 
function paintShape(opts: PaintOptions) {
  // ...
}
 
const shape = getShape();
paintShape({ shape });
paintShape({ shape, xPos: 100 });
paintShape({ shape, yPos: 100 });
paintShape({ shape, xPos: 100, yPos: 100 });
```

在此示例中， `xPos` 和 `yPos` 都被视为可选。我们可以选择提供它们中的任何一个，所以上面对 `paintShape` 的每个调用都是有效的。所有可选性真正说明的是，如果设置了属性，它最好有一个特定的类型。

我们也可以从这些属性中读取 - 但是当我们在 `strictNullChecks` 下读取时，TypeScript 会告诉我们它们可能是 `undefined` 。

![image-20230517142849402](https://s2.loli.net/2023/05/17/HQ1CKMRAWjqmFkb.png)

在 JavaScript 中，即使该属性从未被设置，我们仍然可以访问它——它只会给我们值 `undefined` 。我们可以专门处理 `undefined` 。

![image-20230517142936901](https://s2.loli.net/2023/05/17/jlEAVPUkv4cX59M.png)

请注意，这种为未指定值设置默认值的模式非常普遍，以至于 JavaScript 有语法来支持它。

![image-20230517143014607](https://s2.loli.net/2023/05/17/GO16AginxTwsRQl.png)

这里我们为 `paintShape` 的参数使用了解构模式，并为 `xPos` 和 `yPos` 提供了默认值。现在 `xPos` 和 `yPos` 都肯定存在于 `paintShape` 的主体中，但对于 `paintShape` 的任何调用者都是可选的。

> Note that there is currently no way to place type annotations within destructuring patterns. This is because the following syntax already means something different in JavaScript.
> 请注意，目前无法在解构模式中放置类型注释。这是因为以下语法在 JavaScript 中已经意味着不同的东西。

![image-20230517143500831](https://s2.loli.net/2023/05/17/oC1Ofr8F6M5zqjg.png)

在对象解构模式中， `shape: Shape` 表示“获取属性 `shape` 并将其在本地重新定义为名为 `Shape` 的变量。同样， `xPos: number` 创建了一个名为 `number` 的变量，其值基于参数的 `xPos` 。

使用[映射修饰符](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#mapping-modifiers)，您可以删除 `optional` 属性。

## `readonly`Properties 只读属性

对于 TypeScript，属性也可以标记为 `readonly` 。虽然它不会在运行时更改任何行为，但在类型检查期间不能写入标记为 `readonly` 的属性。

![image-20230517183102268](https://s2.loli.net/2023/05/17/I3mPLaFXeoMsbzH.png)

使用 `readonly` 修饰符并不一定意味着一个值是完全不可变的——或者换句话说，它的内部内容不能改变。这只是意味着属性本身不能被重写。

> 内部属性值可以读取和更新，但是不能被重写。

使用映射修饰符，您可以删除 `readonly` 属性。

## Index Signatures 索引签名

有时您并不能提前知道类型属性的所有名称，但您确实知道值的类型。

在这些情况下，您可以使用索引签名来描述可能值的类型，例如：

![image-20230518102251242](https://s2.loli.net/2023/05/18/FgJtUd7jXCn8QO1.png)

![image-20230518102824914](https://s2.loli.net/2023/05/18/P4fZqDtMgkTWe2o.png)

上面，我们有一个带有索引签名的 `StringArray` 接口。这个索引签名声明当 `StringArray` 被 `number` 索引时，它将返回 `string` 。

索引签名属性只允许使用某些类型： `string` 、 `number` 、 `symbol` 、模板字符串模式和仅包含这些的联合类型。

![image-20230518103750693](https://s2.loli.net/2023/05/18/isOEfBNZ7jWwkaY.png)

可以同时支持两种类型的索引器，但从数值索引器返回的类型必须是从字符串索引器返回的类型的子类型。这是因为当使用数字进行索引时，在索引到对象之前，JavaScript实际上会将其转换为字符串。这意味着使用100(数字)进行索引与使用“100”(字符串)进行索引是相同的，因此两者需要保持一致。

```ts
interface Animal {
  name: string;
}
 
interface Dog extends Animal {
  breed: string;
}
 
// Error: indexing with a numeric string might get you a completely separate type of Animal!
interface NotOkay {
  [x: number]: Animal;
	//'number' index type 'Animal' is not assignable to 'string' index type 'Dog'.
  [x: string]: Dog;
}
```

虽然字符串索引签名是描述“字典”模式的强大方式，但它们还强制所有属性与其返回类型相匹配。这是因为字符串索引声明 `obj.property` 也可用作 `obj["property"]` 。在下面的示例中， `name` 的类型与字符串索引的类型不匹配，类型检查器报错：

![image-20230518103826205](https://s2.loli.net/2023/05/18/Wqkpmzlb1hTPvBF.png)

但是，如果索引签名是属性类型的联合，则可以接受不同类型的属性：

```ts
interface NumberOrStringDictionary {
  [index: string]: number | string;
  length: number; // ok, length is a number
  name: string; // ok, name is a string
}
```

最后，您可以创建索引签名 `readonly` 以防止分配给它们的索引：

![image-20230518111834822](https://s2.loli.net/2023/05/18/GadtnJlhsyiBYSQ.png)

您不能设置 `myArray[2]` 因为索引签名是 `readonly` 。



## Excess Property Checks 额外属性检查

如果传入的参数和定义的不一致，TS会报错。

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
    return {
        color: config.color || "red",
        area: config.width ? config.width * config.width : 20,
    };
}

// let mySquare = createSquare({ colour: "red", width: 100 } as SquareConfig); //强行断言
```

绕过这些检查实际上非常简单。最简单的方法是只使用类型断言：

`{ colour: "red", width: 100 } as SquareConfig`

但是，如果您确定该对象可以具有一些以某种特殊方式使用的额外属性，则更好的方法可能是添加**字符串索引签名**。

```ts
interface SquareConfig {
  color?: string;
  width?: number;
  [propName: string]: any;
}
```

我们稍后会讨论索引签名，但这里我们说 `SquareConfig` 可以有任意数量的属性，只要它们不是 `color` 或 `width` ，它们的类型就无关紧要。

绕过这些检查的最后一种方法（可能有点令人惊讶）是将对象分配给另一个变量：由于分配 `squareOptions` 不会进行过多的属性检查，因此编译器不会给您错误：

```ts
let squareOptions = { colour: "red", width: 100 };
let mySquare2 = createSquare(squareOptions);
```

只要您在 `squareOptions` 和 `SquareConfig` 之间有一个公共属性，上述解决方法就可以工作。在此示例中，它是属性 `width` 。但是，如果变量没有任何公共对象属性，它将失败。例如：

```ts
let squareOptions = { colour: "red" };
let mySquare = createSquare(squareOptions);
// Type '{ colour: string; }' has no properties in common with type 'SquareConfig'.
```

请记住，对于像上面这样的简单代码，您可能不应该试图“绕过”这些检查。对于具有方法和保持状态的更复杂的对象字面量，您可能需要牢记这些技术，但大多数过多的属性错误实际上是一种错误。

## Mapping Modifiers是什么？

在 TypeScript 中，Mapping Modifiers 是一种用于映射类型的修饰符，可以用于创建新的类型。Mapping Modifiers 包括 `readonly`、`partial`、`required` 和 `Pick` 等。

- `readonly`：将所有属性设置为只读属性，即不能修改。
- `partial`：将所有属性设置为可选属性，即可以不传入该属性。
- `required`：将所有属性设置为必填属性，即必须传入该属性。
- `Pick`：从一个类型中选取部分属性，创建一个新的类型。

例如，下面是一个使用 `readonly` 和 `partial` Mapping Modifiers 的示例：

```TS
interface Person {
  readonly name: string;
  age?: number;
}

type ReadonlyPerson = Readonly<Person>; // 将所有属性设置为只读属性
type PartialPerson = Partial<Person>; // 将所有属性设置为可选属性
```

在上面的代码中，我们定义了一个名为 `Person` 的接口，该接口包含一个只读属性 `name` 和一个可选属性 `age`。然后，我们使用 `Readonly` 和 `Partial` Mapping Modifiers 创建了两个新的类型 `ReadonlyPerson` 和 `PartialPerson`，分别将 `Person` 接口中的所有属性设置为只读属性和可选属性。

需要注意的是，Mapping Modifiers 可以组合使用，以创建更复杂的类型。例如，我们可以使用 `Pick` Mapping Modifiers 从一个类型中选取部分属性，然后使用 `Partial` Mapping Modifiers 将这些属性设置为可选属性，例如：

```ts
interface Person {
  name: string;
  age: number;
  address: string;
}

type PersonInfo = Partial<Pick<Person, 'name' | 'age'>>; // 选取 'name' 和 'age' 属性，将其设置为可选属性
```

在上面的代码中，我们定义了一个名为 `Person` 的接口，该接口包含三个属性 `name`、`age` 和 `address`。然后，我们使用 `Pick` Mapping Modifiers 从 `Person` 接口中选取了 `name` 和 `age` 两个属性，然后使用 `Partial` Mapping Modifiers 将这两个属性设置为可选属性，创建了一个新的类型 `PersonInfo`。

这意味着如果您遇到诸如选项包之类的过多属性检查问题，您可能需要修改一些类型声明。在这种情况下，如果可以将同时具有 `color` 或 `colour` 属性的对象传递给 `createSquare` ，则应修改 `SquareConfig` 的定义以反映这一点。

## Extending Types 拓展类型

拥有可能是其他类型的更具体版本的类型是很常见的。例如，我们可能有一个 `BasicAddress` 类型，它描述了在美国发送信件和包裹所必需的字段。

```ts
interface BasicAddress {
  name?: string;
  street: string;
  city: string;
  country: string;
  postalCode: string;
}
```

在某些情况下这就足够了，但如果地址处的建筑物有多个单元，地址通常会有一个与之关联的单元号。然后我们可以描述一个 `AddressWithUnit` 。

```ts
interface AddressWithUnit {
  name?: string;
  unit: string;
  street: string;
  city: string;
  country: string;
  postalCode: string;
}
```

这可以完成工作，但这里的缺点是当我们的更改纯粹是累加时，我们不得不重复 `BasicAddress` 中的所有其他字段。相反，我们可以扩展原始的 `BasicAddress` 类型，并只添加 `AddressWithUnit` 独有的新字段。

```ts
interface BasicAddress {
    name?: string;
    street: string;
    city: string;
    country: string;
    postalCode: string;
}

interface AddressWithUnit extends BasicAddress {
		unit: string;
}
```

`interface` 上的 `extends` 关键字使我们能够有效地从其他命名类型复制成员，并添加我们想要的任何新成员。这对于减少我们必须编写的类型声明样板的数量以及发出信号表明同一属性的多个不同声明可能相关的意图很有用。例如， `AddressWithUnit` 不需要重复 `street` 属性，因为 `street` 源自 `BasicAddress` ，读者会知道这两种类型在某种程度上是相关的。

`interface` 也可以从多个类型上扩展。

```ts
interface Colorful {
  color: string;
}
 
interface Circle {
  radius: number;
}
 
interface ColorfulCircle extends Colorful, Circle {}
 
const cc: ColorfulCircle = {
  color: "red",
  radius: 42,
};
```

## Intersection Types 交叉类型

`interface` 允许我们通过扩展其他类型来构建新类型。 TypeScript 提供了另一种称为交集类型的结构，主要用于组合现有的对象类型。

使用 `&` 运算符定义交集类型。

```ts
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}
 
type ColorfulCircle = Colorful & Circle;
```

```ts
type FlyPig = Bird & Fish & Pig;

let pig: FlyPig = {
	fly: true,
	swim: true,
	eat: true
}
```

在这里，我们将 `Colorful` 和 `Circle` 相交以生成一个包含 `Colorful` 和 `Circle` 的所有成员的新类型。

```ts
function draw(circle: Colorful & Circle) {
  console.log(`Color was ${circle.color}`);
  console.log(`Radius was ${circle.radius}`);
}
 
// okay
draw({ color: "blue", radius: 42 });
 
// oops
draw({ color: "red", raidus: 42 });
// Argument of type '{ color: string; raidus: number; }' is not assignable to parameter of type 'Colorful & Circle'.
// Object literal may only specify known properties, but 'raidus' does not exist in type 'Colorful & Circle'. Did you mean to write 'radius'?
```

## Interfaces vs. Intersections 接口和联合

我们刚刚研究了两种组合类型的方法，它们相似，但实际上有细微差别。通过接口，我们可以使用 `extends` 子句从其他类型进行扩展，并且我们能够对交集执行类似的操作并使用类型别名命名结果。两者之间的主要区别在于冲突的处理方式，这种差异通常是您在接口和交集类型的类型别名之间选择一个而不是另一个的主要原因之一。

## Generic Object Types 通用对象类型

让我们想象一个可以包含任何值的 `Box` 类型 - `string` s、 `number` s、 `Giraffe` s 等等。

```ts
interface Box {
  contents: any;
}
```

现在， `contents` 属性被键入为 `any` ，这有效，但可能会导致事故发生。

我们可以改用 `unknown` ，但这意味着在我们已经知道 `contents` 类型的情况下，我们需要进行预防性检查，或者使用容易出错的类型断言。

```ts
interface Box {
  contents: unknown;
}
 
let x: Box = {
  contents: "hello world",
};
 
// we could check 'x.contents'
if (typeof x.contents === "string") {
  console.log(x.contents.toLowerCase());
}
 
// or we could use a type assertion
console.log((x.contents as string).toLowerCase());
```

一种类型安全的方法是为每种类型的 `contents` 搭建不同的 `Box` 类型。

```ts
interface NumberBox {
  contents: number;
}
 
interface StringBox {
  contents: string;
}
 
interface BooleanBox {
  contents: boolean;
}
```

但这意味着我们必须创建不同的函数或函数重载来对这些类型进行操作。

```ts
function setContents(box: StringBox, newContents: string): void;
function setContents(box: NumberBox, newContents: number): void;
function setContents(box: BooleanBox, newContents: boolean): void;
function setContents(box: { contents: any }, newContents: any) {
  box.contents = newContents;
}
```

相反，我们可以创建一个声明类型参数的通用 `Box` 类型。

```ts
interface Box<Type> {
  contents: Type;
}
```

您可能将其理解为“ `Type`类型 的 `Box` 是其 `contents` 具有 `Type` 类型的东西”。稍后，当我们引用 `Box` 时，我们必须给出一个类型参数来代替 `Type` 。

```ts
let box: Box<string>;
```

将 `Box` 视为真实类型的模板，其中 `Type` 是将被替换为其他类型的占位符。当 TypeScript 看到 `Box<string>` 时，它会将 `Box<Type>` 中的每个 `Type` 实例替换为 `string` ，并最终使用 `{ contents: string }` 之类的东西。换句话说， `Box<string>` 和我们之前的 `StringBox` 的工作方式相同。

![image-20230522181713442](https://s2.loli.net/2023/05/22/37t4bpKvOTqAzCn.png)

`Box` 是可重复使用的，因为 `Type` 可以用任何东西代替。这意味着当我们需要一个新类型的盒子时，我们根本不需要声明一个新的 `Box` 类型（尽管如果我们愿意的话我们当然可以）。

```ts

interface Box<Type> {
	content: Type;
}

interface Apple {
	name: string;
}

type AppleBox = Box<Apple>;

let appleBox: AppleBox = {
	content: {
		name: 'apple'
	}
}
```

这也意味着我们可以通过使用泛型函数来完全避免重载。