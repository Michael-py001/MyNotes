# 20230525-TypeScript基础入门(三)-Type Manipulation

> Type Manipulation是指在TypeScript中操作类型的过程，包括创建新类型、转换现有类型、组合类型等。TypeScript提供了许多类型操作工具，如泛型、交叉类型、联合类型、映射类型、条件类型等，可以帮助开发人员更好地操作类型。

## Creating Types from Types 从类型创建类型

TypeScript 的类型系统非常强大，因为它允许根据其他类型来表达类型。

这个想法的最简单形式是泛型。此外，我们还有各种各样的类型运算符可供使用。也可以根据我们已有的值来表达类型。

通过组合各种类型的运算符，我们可以用简洁、可维护的方式表达复杂的操作和值。在本节中，我们将介绍根据现有类型或值来表达新类型的方法。

- [Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html) - Types which take parameters
  泛型 - 采用参数的类型
- [Keyof Type Operator](https://www.typescriptlang.org/docs/handbook/2/keyof-types.html) - Using the `keyof` operator to create new types
  Keyof 类型运算符 - 使用 `keyof` 运算符创建新类型
- [Typeof Type Operator](https://www.typescriptlang.org/docs/handbook/2/typeof-types.html) - Using the `typeof` operator to create new types
  Typeof 类型运算符 - 使用 `typeof` 运算符创建新类型
- [Indexed Access Types](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html) - Using `Type['a']` syntax to access a subset of a type
  索引访问类型 - 使用 `Type['a']` 语法访问类型的子集
- [Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html) - Types which act like if statements in the type system
  条件类型——在类型系统中表现得像 if 语句的类型
- [Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html) - Creating types by mapping each property in an existing type
  映射类型 - 通过映射现有类型中的每个属性来创建类型
- [Template Literal Types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html) - Mapped types which change properties via template literal strings
  模板文字类型 - 通过模板文字字符串更改属性的映射类型

# Generics泛型

软件工程的一个主要部分是构建不仅具有定义明确且一致的 API，而且还可以重用的组件。能够处理今天和明天的数据的组件将为您提供构建大型软件系统的最灵活的功能。

在 C# 和 Java 等语言中，用于创建可重用组件的工具箱中的主要工具之一是泛型，也就是说，能够创建一个可以处理多种类型而不是单一类型的组件。这允许用户使用这些组件并使用他们自己的类型。

## Hello World of Generics 你好泛型世界

首先，让我们做泛型的“hello world”：恒等函数。 identity 函数是一个函数，它将返回传入的任何内容。您可以将其视为与 `echo` 命令类似的方式。

如果没有泛型，我们要么必须给身份函数一个特定的类型：

```ts
function identity(arg: number): number {
  return arg;
}
```

或者，我们可以使用 `any` 类型描述恒等函数：

```ts
function identity(arg: any): any {
  return arg;
}
```

虽然使用 `any` 肯定是通用的，因为它会导致函数接受 `arg` 类型的任何和所有类型，但实际上我们正在丢失有关函数返回时该类型的信息。如果我们传入一个数字，我们所拥有的唯一信息就是可以返回任何类型。

相反，我们需要一种捕获参数类型的方法，这样我们也可以用它来表示返回的内容。在这里，我们将使用**类型变量**，这是一种特殊类型的变量，适用于类型而不是值。

```ts
function identity<Type>(arg: Type): Type {
  return arg;
}
```

我们现在已经向标识函数添加了一个类型变量 `Type` 。 `Type` 允许我们捕获用户提供的类型（例如 `number` ），以便我们以后可以使用该信息。在这里，我们再次使用 `Type` 作为返回类型。通过检查，我们现在可以看到相同的类型用于参数和返回类型。这允许我们在函数的一侧传输该类型的信息，并在另一侧传输。

我们说这个版本的 `identity` 函数是通用的，因为它适用于一系列类型。与使用 `any` 不同，它与第一个使用数字作为参数和返回类型的 `identity` 函数一样精确（即，它不会丢失任何信息）。

一旦我们编写了通用身份函数，我们就可以通过两种方式之一调用它。第一种方法是将所有参数（包括类型参数）传递给函数：

![image-20230525150333925](https://s2.loli.net/2023/05/25/arVBxONbJI71GCt.png)

在这里，我们明确地将 `Type` 设置为 `string` 作为函数调用的参数之一，在参数周围使用 `<>` 而不是 `()` 表示。

第二种方式也许也是最常见的。这里我们使用类型参数推断——也就是说，我们希望编译器根据我们传入的参数类型自动为我们设置 `Type` 的值：

![image-20230525150437016](https://s2.loli.net/2023/05/25/C1JMzr2Qa8ldpVm.png)

请注意，我们不必在尖括号 ( `<>` ) 中显式传递类型；编译器只查看值 `"myString"` ，并将 `Type` 设置为它的类型。虽然类型参数推断可以成为使代码更短和更易读的有用工具，但当编译器无法推断类型时，您可能需要像我们在上一个示例中所做的那样显式传递类型参数，这在更复杂的示例中可能会发生.

## Working with Generic Type Variables  使用通用类型变量

当您开始使用泛型时，您会注意到当您创建类似 `identity` 的泛型函数时，编译器将强制您在函数主体中正确使用任何泛型类型的参数。也就是说，您实际上将这些参数视为它们可以是任何类型和所有类型。

让我们使用之前的 `identity` 函数：

```ts
function identity<Type>(arg: Type): Type {
  return arg;
}
```

如果我们还想在每次调用时将参数 `arg` 的长度记录到控制台怎么办？我们可能会忍不住这样写：

![image-20230525150756981](https://s2.loli.net/2023/05/25/K4b1nAu9RSqklem.png)

当我们这样做时，编译器会给我们一个错误，告诉我们我们正在使用 `arg` 的 `.length` 成员，但我们在任何地方都没有说过 `arg` 有这个成员。请记住，我们之前说过这些类型变量代表任何类型和所有类型，因此使用此函数的人可能已经传入了 `number` ，它没有 `.length` 成员。

假设我们实际上打算让这个函数直接处理 `Type` 而不是 `Type` 的数组。由于我们正在使用数组， `.length` 成员应该可用。我们可以像创建其他类型的数组一样描述它：

```ts
function loggingIdentity<Type>(arg: Type[]): Type[] {
  console.log(arg.length);
  return arg;
}
```

您可以将 `loggingIdentity` 的类型理解为“泛型函数 `loggingIdentity` 接受一个类型参数 `Type` 和一个参数 `arg` ，它是一个 `Type` 数组，并返回一个 `Type` 数组”如果我们传入一个数字数组，我们将返回一个数字数组，因为 `Type` 将绑定到 `number` 。这允许我们将泛型类型变量 `Type` 用作我们正在使用的类型的一部分，而不是整个类型，从而为我们提供了更大的灵活性。

我们也可以这样编写示例：

```ts
function loggingIdentity<Type>(arg: Array<Type>): Array<Type> {
  console.log(arg.length); // Array has a .length, so no more error
  return arg;
}
```

您可能已经从其他语言中熟悉这种类型的类型。在下一节中，我们将介绍如何创建自己的泛型类型，例如 `Array<Type>` 。

## Generic Types 泛型类型

在前面的部分中，我们创建了适用于一系列类型的泛型函数。在本节中，我们将探讨函数本身的类型以及如何创建泛型接口。

泛型函数的类型和非泛型函数一样，首先列出类型参数，类似于函数声明：

```ts
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: <Type>(arg: Type) => Type = identity;
```

我们也可以为类型中的泛型类型参数使用不同的名称，只要类型变量的数量和类型变量的使用方式一致即可。

```ts
function identity<Input>(arg: Input): Input {
  return arg;
}
 
let myIdentity: <Input>(arg: Input) => Input = identity;
```

我们还可以将泛型类型写成对象字面量类型的调用签名：

```ts
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: { <Type>(arg: Type): Type } = identity;
```

这让我们开始编写我们的第一个通用接口。让我们将前面示例中的对象文字移动到一个接口中：

```ts
interface GenericIdentityFn {
  <Type>(arg: Type): Type;
}
 
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: GenericIdentityFn = identity;
```

在类似的示例中，我们可能希望将通用参数移动为整个接口的参数。这样可以让我们看到我们通用的类型（例如 `Dictionary<string>` 而不仅仅是 `Dictionary` ）。这使得类型参数对接口的所有其他成员可见。

```ts
interface GenericIdentityFn<Type> {
  (arg: Type): Type;
}
 
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: GenericIdentityFn<number> = identity;
```

请注意，我们的示例已经改变，变成了略微不同的内容。我们现在不再描述通用函数，而是有一个非通用函数签名，它是通用类型的一部分。当我们使用 `GenericIdentityFn` 时，现在我们还需要指定相应的类型参数（这里是： `number` ），从而锁定底层调用签名将使用的内容。了解何时将类型参数直接放在调用签名上，何时将其放在接口本身上，将有助于描述类型的哪些方面是通用的。

除了通用接口，我们还可以创建通用类。请注意，无法创建通用枚举和命名空间。

## Generic Classes 泛型类

一个泛型类的结构类似于一个泛型接口。泛型类在类名后面用尖括号（ `<>` ）括起来的泛型类型参数列表。

```ts
class GenericNumber<NumType> {
  "zeroValue": NumType;
  "add": (x: NumType, y: NumType) => NumType;
}
 
let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 2;
myGenericNumber.add = function (x, y) {
  return x + y;
};

let n = myGenericNumber.add(1,2)
console.log("n:", n);
console.log("myGenericNumber.zeroValue:", myGenericNumber.zeroValue);
```

这是对 `GenericNumber` 类的字面上的使用，但您可能已经注意到，没有限制它仅使用 `number` 类型。我们可以使用 `string` 或更复杂的对象。

```ts
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function (x, y) {
  return x + y;
};
 
console.log(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

就像接口一样，将类型参数放在类本身上，让我们确保类的所有属性都使用相同的类型。

正如我们在“类”一节中所介绍的那样，类的类型有两个方面：静态方面和实例方面。泛型类仅在其实例方面上是泛型的，而不是在其静态方面上，因此在处理类时，静态成员不能使用类的类型参数。

泛型类仅在其实例方面上是泛型的，而不是在其静态方面上。这意味着，静态成员不能使用类的类型参数。例如，在 `GenericNumber` 类中，`zeroValue` 和 `add` 是静态成员，它们不能使用 `NumType` 类型参数。因此，在处理类时，您需要注意这一点，并确保您的静态成员不依赖于类的类型参数。

## Generic Constraints 泛型约束

如果您还记得之前的例子，您可能有时想编写一个通用函数，该函数适用于一组类型，您对该类型集合将具有哪些功能有一些了解。在我们的 `loggingIdentity` 示例中，我们希望能够访问 `arg` 的 `.length` 属性，但编译器无法证明每种类型都有 `.length` 属性，因此它警告我们不能做出这种假设。

![image-20230525164352734](https://s2.loli.net/2023/05/25/WEySU7LlstAxpwY.png)



与其使用任何和所有类型，我们希望将此函数限制为仅使用具有 `.length` 属性的任何和所有类型。只要类型具有此成员，我们将允许它，但必须至少具有此成员。为此，我们必须将我们的要求列为对 `Type` 可以是什么的约束。

为此，我们将创建一个描述我们约束条件的接口。在这里，我们将创建一个具有单个 `.length` 属性的接口，然后我们将使用此接口和 `extends` 关键字来表示我们的约束条件：

```ts
interface hasLength { 
	length: number;
}

function loggingIdentity4<Type extends hasLength >(arg: Type ):Type {
	console.log(arg.length); //Now we know it has a .length property, so no more error
	return arg
}
```

由于泛型函数现在受到限制，它将不再适用于任何类型。

![image-20230525164543431](https://s2.loli.net/2023/05/25/RME4LgQhYq3W8AB.png)

相反，我们需要传入具有所有必需属性的类型的值：

```ts
loggingIdentity({ length: 10, value: 3 });
```

## Using Type Parameters in Generic Constraints 在泛型约束中使用类型参数

您可以声明一个由另一个类型参数约束的类型参数。例如，这里我们想通过名称从对象中获取属性。我们希望确保不会意外地获取不存在于 `obj` 上的属性，因此我们将在两个类型之间放置一个约束：

![image-20230525165830846](https://s2.loli.net/2023/05/25/URWO1MAw6EnZTu5.png)

```ts

let arr6 = {a:1,b:2,c:3}
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
	return obj[key];
} 

let x = getProperty(arr6, "a");
// let y = getProperty(arr6, "d"); //报错
```

## Using Class Types in Generics 在泛型中使用类类型

在使用泛型创建TypeScript工厂时，需要通过它们的构造函数引用类类型。例如，

```ts
function create<Type>(c: { new (): Type }): Type {
  return new c();
}
```

这个函数名为 `create`，它是一个泛型函数，接受一个参数 `c`，类型为一个构造函数，返回值类型为 `Type`。

这个函数的作用是创建一个类型为 `Type` 的对象。它通过调用 `c` 构造函数来创建一个新的对象，并将其返回。由于 `c` 是一个构造函数，因此它可以用来创建任何类型的对象，只要这些类型具有无参构造函数。

例如，如果有一个类 `Person`，它有一个无参构造函数，那么可以使用 `create` 函数来创建一个 `Person` 类型的对象：

```ts
class Person {
  name: string;
  age: number;
  constructor() {
    this.name = "Alice";
    this.age = 30;
  }
}

const person = create(Person); // person 的类型为 Person
console.log(person); // 输出：Person { name: 'Alice', age: 30 }
```

在上面的例子中，`create(Person)` 返回一个类型为 `Person` 的对象，它的 `name` 属性为 `"Alice"`，`age` 属性为 `30`。



一个更高级的示例使用原型属性来推断和约束构造函数和类类型实例之间的关系。

```ts
class BeeKeeper {
  hasMask: boolean = true;
}
 
class ZooKeeper {
  nametag: string = "Mikle";
}
 
class Animal {
  numLegs: number = 4;
}
 
class Bee extends Animal {
  keeper: BeeKeeper = new BeeKeeper();
}
 
class Lion extends Animal {
  keeper: ZooKeeper = new ZooKeeper();
}
 
function createInstance<A extends Animal>(c: new () => A): A {
  return new c();
}
 
createInstance(Lion).keeper.nametag;
createInstance(Bee).keeper.hasMask;
```

## Generic Parameter Defaults 泛型参数默认值

考虑一个创建新 `HTMLElement` 的函数。不带参数调用该函数会生成 `Div` ；将元素作为第一个参数调用它会生成该参数类型的元素。您还可以选择传递一个子元素列表。以前，您必须将其定义为：

```ts
/* 泛型默认值 */
declare function create2(): Container<HTMLDivElement, HTMLDivElement[]>;

declare function create3<T extends HTMLElement>(element: T): Container<T, T[]>;

declare function create4<T extends HTMLElement, U extends HTMLElement>(
	element: T,
	children: U[]
): Container<T, U>;
```

使用通用参数默认值，我们可以将其简化为：

```ts
declare function create5<T extends HTMLElement = HTMLDivElement, U = T[]>(
	element?: T,
	children?: U
): Container<T, U>;
```

泛型参数默认值遵循以下规则：

1. 如果类型参数具有默认值，则被视为可选。
2. 必选类型参数不能在可选类型参数之后。
3. 如果存在类型参数的约束条件，那么类型参数的默认类型必须符合该约束条件。
4. 在指定类型参数时，您只需要为必选的类型参数指定类型参数。未指定的类型参数将解析为它们的默认类型。
5. 如果指定了默认类型，但推断无法选择候选项，则会推断出默认类型。
6. 与现有的类或接口声明合并的类或接口声明可以为现有的类型参数引入默认类型。
7. 只要指定了默认值，与现有类或接口声明合并的类或接口声明可以引入新的类型参数。

