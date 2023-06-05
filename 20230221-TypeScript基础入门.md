# 20230221-TypeScript基础入门

## 安装

### 1.全局安装 typescript

```
npm install -g typescript
```

安装后cmd输入tsc -v 可查看版本号，此时可以运行 tsc xxx.ts 把xxx文件编译成xxx.js文件。

### 2. 全局安装ts-node

```
npm install -g ts-node
```

安装它的原因是typescript自带的tsc命令并不能直接运行typescript代码，它封装了typescript的编译过程，提供直接运行typescript代码的能力。

# TS与JS中的类型

JavaScript 中已经有一些基本类型可用：`boolean`、 `bigint`、 `null`、`number`、 `string`、 `symbol `和 `undefined`，引用数据类型：`object`，一共8种， 它们都可以在接口中使用。TypeScript 将此列表扩展为更多的内容，例如:

- `any` （允许任何类型）
- [`unknown`](https://www.typescriptlang.org/play#example/unknown-and-never) （确保使用此类型的人声明类型是什么）
-  [`never`](https://www.typescriptlang.org/play#example/unknown-and-never) （这种类型不可能发生）
-  `void` （返回 `undefined` 或没有返回值的函数）。

构建类型有两种语法： [接口和类型](https://www.typescriptlang.org/play/?e=83#example/types-vs-interfaces)。 你应该更喜欢 `interface`。当需要特定功能时使用 `type` 。

### Other important TypeScript types

| Type           | Explanation                                   |
| :------------- | :-------------------------------------------- |
| `unknown`      | the top type.                                 |
| `never`        | the bottom type.                              |
| object literal | eg `{ property: Type }`                       |
| `void`         | for functions with no documented return value |
| `T[]`          | mutable arrays, also written `Array<T>`       |
| `[T, T]`       | tuples, which are fixed-length but mutable    |
| `(t: T) => U`  | functions                                     |

# 定义类型

要创建具有推断类型的对象，该类型包括 `name: string` 和 `id: number`，你可以这么写：

```typescript
const user = {
  name: "Hayes",
  id: 0,
};
```

![image-20230222154813566](https://s2.loli.net/2023/02/22/5zSvFLiaynOYlkM.png)

## 变量类型注解

当你使用 `const` 、 `var` 或 `let` 声明变量时，您可以选择添加类型注释以显式指定变量的类型：

```ts
let myName: string = "Alice";
```

## 函数中的类型注解

### 函数返回值类型注解

您可以添加返回类型注释。返回类型注释出现在参数列表之后：

```ts
function getFavoriteNumber(): number {
  return 26;
}
```

### 函数参数类型注解

当你声明一个函数时，你可以在每个参数后面加上类型注解来声明函数接受什么类型的参数。参数类型注释位于参数名称之后：

```ts
function greet(name: string) {
  console.log("Hello, " + name.toUpperCase() + "!!");
}
```

![image-20230504164525348](https://s2.loli.net/2023/05/04/JAnh5yPmIgrzbfR.png)

### 匿名函数

匿名函数与函数声明有点不同。当一个函数出现在 TypeScript 可以确定如何调用它的地方时，该函数的参数会自动指定类型。

![image-20230504165957281](https://s2.loli.net/2023/05/04/LIE3sZeCgkPupBK.png)

即使参数 `s` 没有类型注释，TypeScript 使用 `forEach` 函数的类型以及数组的推断类型来确定 `s` 将具有的类型。

即使参数 `s` 没有类型注释，TypeScript 使用 `forEach` 函数的类型以及数组的推断类型来确定 `s` 将具有的类型。

此过程称为上下文类型化，因为函数出现的上下文告知它应该具有什么类型。与推理规则类似，您无需明确了解这是如何发生的，但它确实会发生可以帮助您注意到何时不需要类型注释

## 对象类型注解

在这里，我们用具有两个属性的类型注释参数 - `x` 和 `y` - 都是 `number` 类型。

每个属性的类型部分也是可选的。如果您不指定类型，它将被假定为 `any` 

![image-20230504170454881](https://s2.loli.net/2023/05/04/NxizVy3lDMLCcWS.png)

## 可选属性

对象类型还可以指定它们的部分或全部属性是可选的。为此，请在属性名称后添加一个 `?` 

![image-20230504170626506](https://s2.loli.net/2023/05/04/h1qRCcWdXZNUzYt.png)

在 JavaScript 中，如果您访问不存在的属性，您将获得值 `undefined` 而不是运行时错误。因此，当您从可选属性中读取时，您必须在使用它之前检查 `undefined` 。

![image-20230504171107157](https://s2.loli.net/2023/05/04/lCTAdt3Osf7aSKz.png)

需要加上判断，确保last有值

![image-20230504171216680](https://s2.loli.net/2023/05/04/GxV1vynNirwUZmL.png)

使用可选链操作符也可以避免报错：

![image-20230504171312561](https://s2.loli.net/2023/05/04/OGvVLal1fDuH7Kt.png)

# Literal Types 字面量类型

除了一般类型 `string` 和 `number` ，我们还可以在类型位置引用特定的字符串和数字。

考虑这一点的一种方法是考虑 JavaScript 如何以不同的方式来声明变量。 `var` 和 `let` 都允许更改变量中保存的内容，而 `const` 则不允许。这反映在 TypeScript 如何为文字创建类型上。

![image-20230505104253766](https://s2.loli.net/2023/05/05/qBVlHTSUQOpt9kf.png)

就其本身而言，文字类型不是很有价值：

![image-20230505104719069](https://s2.loli.net/2023/05/05/UMmZ8krjHqtATzO.png)

有一个只能有一个值的变量并没有什么用处! 

但是，通过将字词组合成联合体，你可以表达一个更有用的概念--例如，只接受某一组已知值的函数：

![image-20230505104804707](https://s2.loli.net/2023/05/05/9t4AU12PrJR8EIQ.png)

![image-20230505104833308](https://s2.loli.net/2023/05/05/aMWNHBjcl5VK8bh.png)

![image-20230505104841398](https://s2.loli.net/2023/05/05/djHFStIxlvPi3cr.png)

# Literal Inference 字面量推断

当你用一个对象初始化一个变量时，TypeScript 假设该对象的属性以后可能会改变值。例如，如果你写了这样的代码：

```ts
const obj = { counter: 0 };
if (someCondition) {
  obj.counter = 1;
}
```

TypeScript不会认为将1赋值给之前为0的字段是一个错误。另一种说法是，obj.counter必须有数字类型，而不是0，因为类型是用来决定读写行为的。

```ts
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.
```

在上面的例子中，req.method被推断为字符串，而不是 "GET"。因为在创建req和调用handleRequest之间可以评估代码，这可能会给req.method分配一个新的字符串，如 "GUESS"，TypeScript认为这段代码有一个错误。❗

有两种方法可以解决这个问题。

1. 你可以通过在任一位置添加一个类型断言来改变推论：
   ![image-20230505105939524](https://s2.loli.net/2023/05/05/axcF8BOvIXy3DM7.png)

​	变化1意味着 "我打算让req.method总是有字面类型 "GET""，防止之后可能的 "GUESS "分配到该字段。变化2意味着 "由于其他原因，我知道req.method具有

2. 你可以使用`as const`来将整个对象转换为类型字：

   ```ts
   const req = { url: "https://example.com", method: "GET" } as const;
   handleRequest(req.url, req.method);
   ```

 `as const` 后缀的作用类似于const，但对类型系统而言，确保所有属性都被分配到字面类型，而不是像字符串或数字这样的更一般的版本。

# 类型别名

我们通过直接在类型注解中编写对象类型和联合类型来使用它们。

## 用type定义类型

![image-20230504175141358](https://s2.loli.net/2023/05/04/1eJhZ5E3aKgqANs.png)

```js
type BirdType = {
  wings: 2;
};

const bird1: BirdType = { wings: 2 };
```

## 用interface定义类型 

*接口声明* 是命名对象类型的另一种方式

![image-20230504175611476](https://s2.loli.net/2023/05/04/BRLgE7bP5yoNInW.png)

### 定义接口声明

你可以使用 `interface` 关键字声明显式地描述此对象的*内部数据的类型*

```typescript
interface User {
  name: string;
  id: number;
}
```

### 接口声明与类一起使用

然后你可以声明一个符合此接口（`interface`）的 JavaScript 对象，在变量声明后使用像 `: TypeName` 这样的语法：

```typescript
interface User {
  name: string;
  id: number;
}
class UserAccount {
  name:string;
  id:number;

  constructor(name:string, id:number) {
    this.name = name;
    this.id = id
  }
}

const userOne: User = new UserAccount('Lili',1)
```

### 定义函数返回值类型

```typescript
interface User {
  name:string;
  id:number;
}
function getUser(): User {
  return {
    name: 'admin',
    id:2
  }
}
```



### 定义函数参数类型

```typescript
interface User {
  name:string;
  id:number;
}
function deleteUser(user: User) {
  // ...
}
deleteUser({
  name: 'admin',
  id: 0
})
```

# type 和interface 定义类型对比

类型别名和接口非常相似，在大多数情况下你可以在它们之间自由选择。 几乎所有的 `interface` 功能都可以在 `type` 中使用，关键区别在于不能重新开放类型以添加新的属性，而接口始终是可扩展的。

- Interface拓展接口

  ```ts
  interface Animal {
    name: string
  }
  
  interface Bear extends Animal {
    honey: boolean
  }
  
  const bear = getBear() 
  bear.name
  bear.honey
  ```

  向现有接口添加新字段

  ```ts
  interface Window {
    title: string
  }
  
  interface Window {
    ts: TypeScriptAPI
  }
  
  const src = 'const a = "Hello World"';
  window.ts.transpileModule(src, {});
  ```

  

  - **Type**拓展接口

    ```ts
    type Animal = {
      name: string
    }
    
    type Bear = Animal & { 
      honey: Boolean 
    }
    
    const bear = getBear();
    bear.name;
    bear.honey;
    ```

    类型创建后不能更改

    ```ts
    type Window = {
      title: string
    }
    
    type Window = {
      ts: TypeScriptAPI
    }
    
     // Error: Duplicate identifier 'Window'.
    ```


## 给Window拓展属性

```ts
interface Window {
  title: string
}

interface TypeScriptAPI {
  version: string
  transpileModule: (input: string, compilerOptions: any) => string
}

interface Window {
  ts: TypeScriptAPI
}

window.ts = {
  version : '1.0.0',
  transpileModule : (input: string, compilerOptions: any) => {
    return input
  }
}

console.log("window:",window.ts.version);
```



1. interface和type定义的类型可以相互使用，没有差别

2. 他们都支持拓展其他类型或接口，Type通过`&`符号并上其他类型，interface则通过`extends`关键字继承其他接口
3. 推荐使用interface,因为可以提供更好的错误信息

```typescript
// There are two main tools to declare the shape of an
// object: interfaces and type aliases.
//
// They are very similar, and for the most common cases
// act the same.
type BirdType = {
  wings: 2;
};

interface BirdInterface {
  wings: 2;
}

const bird1: BirdType = { wings: 2 };
const bird2: BirdInterface = { wings: 2 };

// Because TypeScript is a structural(结构性) type system,
// it's possible to intermix(相互混合) their use too.
// 1、interface和type定义的类型可以相互使用，没有差别
const bird3: BirdInterface = bird1;

// They both support extending other interfaces and types.
// Type aliases do this via(通过) intersection(& 相交符号) types, while
// interfaces have a keyword(extends).
// 2. 他们都支持拓展其他类型或接口，Type通过`&`符号并上其他类型，interface则通过`extends`关键字继承其他接口
type Owl = { nocturnal: true } & BirdType;
type Robin = { nocturnal: false } & BirdInterface;

interface Peacock extends BirdType {
  colourful: true;
  flies: false;
}
interface Chicken extends BirdInterface {
  colourful: false;
  flies: false;
}

let owl: Owl = { wings: 2, nocturnal: true };
let chicken: Chicken = { wings: 2, colourful: false, flies: false };

// That said, we recommend you use interfaces over type
// aliases. Specifically, because you will get better error
// messages. If you hover over the following errors, you can
// see how TypeScript can provide terser(精炼的) and more focused(更专注)
// messages when working with interfaces like Chicken.
//3. 推荐使用interface,因为可以提供更好的错误信息
owl = chicken;
chicken = owl;
```

assignable  可赋值的

![image-20230222185003400](https://s2.loli.net/2023/02/22/XCTxEZUGsRwvlto.png)

![image-20230222185016946](https://s2.loli.net/2023/02/22/C5VM4xAHpyiXe6K.png)

# 组合类型

> [TypeScript: 组合类型](https://www.typescriptlang.org/zh/docs/handbook/typescript-in-5-minutes.html#组合类型)

使用 TypeScript，可以通过组合简单类型来创建复杂类型。有两种流行的方法可以做到这一点：**联合和泛型**。

## Union Types 联合类型

使用联合，可以声明类型可以是许多类型中的一种。例如，可以将 `boolean` 类型描述为 `true` 或 `false` ：

```ts
type MyBool = true | false;
```

注意：_如果将鼠标悬停在上面的 `MyBool` 上，您将看到它被归类为 `boolean`。这是结构化类型系统的一个属性。下面有更加详细的信息。

![image-20230223115905248](https://s2.loli.net/2023/02/23/GVhcmNdUkxCEfQ4.png)

你可以传入联合类型中的其中之一一种属性，但是不能传入联合类型以外的属性

![image-20230504172831070](https://s2.loli.net/2023/05/04/m96CzEQh3xbdtGe.png)

联合类型的一个流行用法是描述 `string` 或者 `number` 的[字面量](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types)的合法值。

```typescript
type WindowStates = "open" | "closed" | "minimized";
type LockStates = "locked" | "unlocked";
type PositiveOddNumbersUnderTen = 1 | 3 | 5 | 7 | 9;
```

联合也提供了一种处理不同类型的方法。例如，可能有一个函数处理 `array` 或者 `string`：

```typescript
function getLength(obj: string | string[]) {
  return obj.length;
}
```

你可以使函数根据传递的是字符串还是数组返回不同的值：

```typescript
function wrapInArray(obj: string | string[]) {
  if (typeof obj === "string") {
    return [obj];
            
(parameter) obj: string
  }
  return obj;
}
```

### 案例

TS中的联合类型（Union Types）可以用于指定一个变量可以是多种类型中的任何一种，即多个类型中的一个。

语法上，使用 "|" 符号将多个类型连接起来，例如：

```typescript
let myVar: number | boolean;
myVar = 10;  // 合法
myVar = true;  // 合法
myVar = "hello";  // 报错，不是指定的类型之一
```

上面的例子中，变量 myVar 的类型被定义为 "number" 或 "boolean" 类型中的一种，可以存储任意一个类型的值。如果尝试存储其他类型，则会出现类型错误。

### 案例二

```ts
type D = string | number;

function printValue(value: D) {
  console.log(value);
}

printValue("Hello"); // 输出：Hello
printValue(42); // 输出：42
```

在上述代码中，`D` 类型是由 `string` 和 `number` 两个类型通过 `|` 符号组合而成的。 `printValue()` 函数接受一个 `D` 类型的参数，因此可以接受 `string` 或 `number` 类型的值作为参数。

需要注意的是，联合类型中的值只能是其中一个类型中的值，而交叉类型中的值必须同时满足所有类型的要求。例如，类型 `string | number` 中的值可以是字符串或数字中的任意一种，而类型 `A & B` 中的值必须同时包含 `A` 和 `B` 中的所有属性和方法。

TypeScript 只允许你对联合体做一些事情，前提是联合体的每个成员都有效。例如，如果你有联合 `string | number` ，你就不能使用仅在 `string` 上可用的方法：

![image-20230504173425244](https://s2.loli.net/2023/05/04/ZBU8qLuhsNIcmjW.png)

解决方案是通过代码缩小联合，就像在没有类型注释的 JavaScript 中一样。当 TypeScript 可以根据代码的结构为值推断出更具体的类型时，就会发生缩小。

例如，TypeScript 知道只有 `string` 值才会有 `typeof` 值 `"string"` ：

![image-20230504173537787](https://s2.loli.net/2023/05/04/im3HygJwl9Vk4YI.png)

![image-20230504173933905](https://s2.loli.net/2023/05/04/ADONKbwMPfUul21.png)

联合类型在开发中非常有用，可以在需要支持多种类型的场景下使用，避免使用 any 类型或其他不符合规范的做法。同时，在使用联合类型的时候，还可以使用类型保护技术，例如 typeof 或 instanceof 等，以提高代码的健壮性和可读性。

## 泛型

在TypeScript 中，**泛型（Generics）**是一种可重用代码的技巧，它可以创建能够用于不同类型的代码组件。**所谓泛型，就是指在定义函数、接口或类的时候，不预先指定具体类型，而是在使用的时候再指定类型参数的一种特性。**

通过泛型，可以使函数、接口或类的形参类型与返回值的类型变得灵活，并且可以由开发者在使用时进行指定；同时可以很好地防止类型错误。

下面是一个使用泛型的简单例子：

```typescript
function Identity<T>(arg: T): T {
  return arg;
}

let output1 = Identity<string>("Hello World!");
let output2 = Identity<number>(1000);

console.log(output1);
console.log(output2);
```

在这个例子中，我们定义了一个Identity的泛型函数，使用T作为**类型变量**。我们在函数的参数里使用了这个**T类型变量**，并且**返回了一个T类型变量**。 调用这个泛型函数时，需要在函数名后面使用尖括号括起来指定泛型类型。

这个例子中的泛型函数可以处理任何类型的参数并返回该类型，因此我们**传入什么类型的参数，就会返回相同的类型的结果。**

通过泛型，我们可以编写灵活、可重用的代码，提高代码的健壮性和可读性。

### 案例

**泛型为类型提供变量**。一个常见的例子是数组。没有泛型的数组可以包含任何内容。带有泛型的数组可以描述数组包含的值。

```typescript
type StringArray = Array<string>;
type NumberArray = Array<number>;
type ObjectWithNameArray = Array<{ name: string }>;
```

你可以声明自己使用泛型的类型：

```typescript
interface Backpack<Type> {
  add: (obj: Type) => void;
  get: () => Type;
}
 
// 这一行是一个简写，可以告诉 TypeScript 有一个常量，叫做`backpack`，并且不用担心它是从哪
// 里来的。
declare const backpack: Backpack<string>;
 
// 对象是一个字符串，因为我们在上面声明了它作为 Backpack 的变量部分。
const object = backpack.get();
 
// 因为 backpack 变量是一个字符串，不能将数字传递给 add 函数。
backpack.add(23);
```

![image-20230302184859995](https://s2.loli.net/2023/03/02/pgJWKzaUxeq2oLN.png)

# Enums枚举

Enums是TypeScript添加到JavaScript的一个特性，它允许描述一个值，这个值可以是一组可能的命名常量中的一个。与大多数TypeScript特性不同，这不是对JavaScript的类型级添加，而是添加到语言和运行时的东西。正因为如此，你应该知道这个特性的存在，但除非你确定，否则可以暂不使用。你可以在[枚举参考页](https://www.typescriptlang.org/docs/handbook/enums.html)上阅读更多关于枚举的信息。

# Narrowing TS检测类型范围缩小

`string.repeat()`，传入的参数只能是number类型，而padLeft的padding参数是number或string类型，此时ts会报错。

![image-20230505162603664](https://s2.loli.net/2023/05/05/IGrxkXApM8y1DaN.png)

需要事先检查padding的类型：

![image-20230505163213321](https://s2.loli.net/2023/05/05/oxCOXuj6yBdqRhJ.png)

以上代码我们可以理解为一种类型保护的特殊形式的代码。将类型细化比声明的更具体的类型的过程成为缩小。

# type guards 类型保护

正如我们所看到的，JavaScript 支持一个 `typeof` 运算符，它可以返回一个值的基本类型信息，比如：

- `"string"`
- `"number"`
- `"bigint"`
- `"boolean"`
- `"symbol"`
- `"undefined"`
- `"object"`
- `"function"`

在 TypeScript 中，检查 `typeof` 返回的值是一种类型保护。因为 TypeScript 对 `typeof` 对不同值的操作方式进行了编码，所以它知道它在 JavaScript 中的一些怪癖。例如，请注意在上面的列表中， `typeof` 不返回字符串 `null` 。在 JavaScript 中， `typeof null` 实际上是 `"object"`。

TypeScript 让我们知道 `strs` 只缩小到 `string[] | null` ，而不仅仅是 `string[]`

![image-20230505171527477](https://s2.loli.net/2023/05/05/ynm8SW7PQxBjc3C.png)

# `never` 类型

缩小范围时，您可以将联合的选项减少到您已经删除了所有可能性并且没有任何东西的点。在这些情况下，TypeScript 将使用 `never` 类型来表示不应存在的状态。

`never` 类型可分配给每个类型;但是，没有类型可以分配给 `never` （ `never` 本身除外）。这意味着您可以使用缩小并依靠 `never` 打开在 switch 语句中进行详尽检查。

例如，将 `default` 添加到我们的 `getArea` 函数中，该函数尝试将形状分配给 `never` 将在未处理所有可能的情况时引发。

将新成员添加到 `Shape` 联合将导致 TypeScript 错误：

![image-20230505172708695](https://s2.loli.net/2023/05/05/FWbSRNOQdJyK4l7.png)

# 结构化的类型系统

TypeScript 的一个核心原则是类型检查基于对象的属性和行为（type checking focuses on the *shape* that values have）。这有时被叫做“鸭子类型”或“结构类型”（structural typing）。

在结构化的类型系统当中，如果两个对象具有相同的结构，则认为它们是相同类型的。

```typescript
interface Point {
  x: number;
  y: number;
}
 
function logPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`);
}
 
// 打印 "12, 26"
const point = { x: 12, y: 26 };
logPoint(point);
```

`point` 变量从未声明为 `Point` 类型。 但是，在类型检查中，TypeScript 将 `point` 的结构与 `Point`的结构进行比较。它们的结构相同，所以代码通过了。

- **结构匹配只需要匹配对象字段的子集。**

![image-20230308093028250](https://s2.loli.net/2023/03/08/uQpG3PgcDRBl1qn.png)

- **类和对象确定结构的方式没有区别：**

  ```typescript
  interface Point {
    x: number;
    y: number;
  }
   
  function logPoint(p: Point) {
    console.log(`${p.x}, ${p.y}`);
  }
  // ---分割线---
  class VirtualPoint {
    x: number;
    y: number;
   
    constructor(x: number, y: number) {
      this.x = x;
      this.y = y;
    }
  }
   
  const newVPoint = new VirtualPoint(13, 56);
  logPoint(newVPoint); // 打印 "13, 56"
  ```

  如果对象或类具有所有必需的属性，则 TypeScript 将表示是它们匹配的，而不关注其实现细节。

# Type Assertions 类型断言

从 TypeScript 1.6 版本开始，TypeScript 提供了类型断言（Type Assertion）机制，可以用来手动指定变量的类型，以适应开发者的特定需求。

有时你会得到 TypeScript 不知道的值的类型信息。

例如，如果您使用 `document.getElementById` ，TypeScript 只知道这将返回某种 `HTMLElement` ，但您可能知道您的页面将始终有一个带有给定 ID 的 `HTMLCanvasElement` 。

在这种情况下，您可以使用类型断言来指定更具体的类型：

```ts
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```

与类型注释一样，类型断言会被编译器删除，不会影响代码的运行时行为。

您还可以使用尖括号语法（除非代码在 `.tsx` 文件中），这是等效的：

```ts
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

提醒：因为类型断言在编译时被移除，所以没有与类型断言关联的运行时检查。如果类型断言错误，则不会出现异常或生成 `null` 。

类型断言有两种语法形式：

1. 尖括号语法

   ```typescript
   let myVar: any = "Hello, TypeScript!";
   let myStr: string = (<string>myVar);
   console.log(myStr.length);  // 合法，输出 17
   ```

2. as 语法

   ```typescript
   let myVar: any = "Hello, TypeScript!";
   let myStr: string = (myVar as string);
   console.log(myStr.length);  // 合法，输出 17
   ```

TypeScript 只允许转换为更具体或更不具体的类型版本的类型断言。此规则可防止“不可能的”强制转换，例如：

![image-20230504185400783](https://s2.loli.net/2023/05/04/gXUa7rvAhkVMt1Y.png)

有时此规则可能过于保守，并且会禁止可能有效的更复杂的强制转换。如果发生这种情况，您可以使用两个断言，首先是 `any` （或 `unknown` ，我们稍后会介绍），然后是所需的类型：

![image-20230504185423682](https://s2.loli.net/2023/05/04/fdwsJ4rQIlR8caD.png)

## 强行断言

强制类型转换，也叫作“类型断言”（Type Assertion），它属于一种人为干预类型检查的做法。开发者在使用类型断言时，必须手动指定变量的类型，以适应特定的开发需求。

有时候，在进行类型检查时，TypeScript 为避免类型错误，会在变量赋值时给出警告。但是，如果开发者认为这种警告可以被忽略，因为其知道变量的确切类型，那么就可以使用类型强制转换来“打消”TypeScript的警告。

强制类型转换的符号有两种语法形式：

1. 尖括号语法

   ```typescript
   let obj: any = 'hello';
   let strLength: number = (<string>obj).length;
   ```

2. as关键字语法

   ```typescript
   let obj: any = 'hello';
   let strLength: number = (obj as string).length;
   ```

需要注意的是，对于类型强制转换的使用要保证变量的类型确实是可以断言的，否则就可能引发程序错误。

并且， TypeSript使用类型断言来改变数据类型的本质并没有发生变化，变量的类型仍然是该变量在编译期按照TypeScript的类型推断规则所推断出，而不是由TypeScript 强制指定的类型。因此，它并不能真正的改变变量的类型，只是为了规避TypeScript的类型检查而已。

# 数组类型的声明

比如一下这段代码：

```ts
function getLength(obj: string | string[]) {
  return obj.length;
}
```

其中`obj: string | string[]`这段代码的意思是：

函数 `getLength(obj: string | string[])` 的参数类型定义中，`string | string[]` 表示 obj 可以是一个字符串或者字符串数组。

也就是说，这个函数可以接受两种类型的参数：字符串或者字符串数组。

### 什么是字符串数组？

字符串数组是由多个字符串组成的数组。可以使用 TypeScript 中的数组类型声明来表示字符串数组。比如，下面是一个示例：

```ts
const myArray: string[] = ['apple', 'banana', 'orange'];
```

在上面的示例中，`myArray` 是一个类型为 `string[]` 的数组，它包含了三个字符串元素 `'apple'`、`'banana'` 和 `'orange'`。

通过这种方式，我们可以将多个字符串存储在一个数组中，以便于组织和处理。在 TypeScript 中，数组类型是一种非常重要的类型，我们经常会用到它来存储一组相关的数据。

### TS中所有类型的数组有哪些？

1. **数字数组**：由多个数字组成的数组，可以使用 `number[]` 类型声明表示，例如 `[1, 2, 3]`。
2. **布尔数组**：由多个布尔值组成的数组，可以使用 `boolean[]` 类型声明表示，例如 `[true, false, true]`。
3. **对象数组**：由多个对象组成的数组，可以使用对象类型声明的数组表示，例如 `{name: string, age: number}[]` 表示由多个拥有 `name` 和 `age` 属性的对象组成的数组。
4. **任意类型数组**：由多种类型组成的数组，可以使用 `any[]` 类型声明表示，例如 `['apple', 2, true]`，其中包含了字符串、数字和布尔类型。

除此之外，TypeScript 还支持**元组类型（tuple）**，它可以限定数组中每个元素的类型和数量。元组类型使用 `[type1, type2, ...]` 这样的语法来定义。例如，`[string, number]` 表示只包含两个元素，第一个元素为字符串类型，第二个元素为数字类型的数组。

# declare语法

案例代码：`declare type ConfigDuration = number | (() => void);`

declare是TypeScript中的类型声明语法，用于声明一个类型别名。具体来说，declare type ConfigDuration = number | (() => void) 声明了一个类型别名 ConfigDuration，它的值可以是数字或者一个返回值为 void 的函数。

除了用于声明类型别名外，declare 的语法还包括：

1. 声明全局变量或函数：declare var、declare function
2. 声明全局命名空间：declare namespace
3. 声明模块：declare module
4. 声明类、接口、枚举等：declare class、declare interface、declare enum

这些语法都是用于在TypeScript中描述JavaScript代码结构的特殊语法。

```ts
declare type JointContent = VNodeTypes | MessageArgsProps;
export declare type ConfigOnClose = () => void;
```

# () => void 是什么含义？

`() => void` 是一个函数类型的字面量，表示这是一个不需要参数，返回值为 `void` 的函数类型。

在 TypeScript 中，`void` 是一个特殊的类型，表示没有返回值的函数或表达式。例如：

```ts
function sayHello(): void {
  console.log("Hello!");
}

const result: void = sayHello();
console.log(result); // undefined
```

在上述代码中，`sayHello()` 函数的返回值是 `void`，也就是没有返回值。因此，`result` 的值为 `undefined`，因为它实际上没有值。 `void` 可以用于函数的返回值类型、变量的类型注解等场景。

# interface 、type 、declare类型声明的语法有什么区别？

`interface` 和 `type` 也是用于类型声明的语法，但它们与 `declare` 有一些区别。

- `interface` 用于声明对象类型、类类型和函数类型等，可以用于描述对象的结构和行为，也可以用于进行类型检查。例如：

```ts
interface Person {
  name: string;
  age: number;
  sayHello(): void;
}
```

- `type` 用于声明类型别名，可以将一个类型命名为另一个名称，增加代码可读性和复用性。例如：

  ```ts
  type ConfigDuration = number | (() => void);
  ```

相对于 `declare`，`interface` 和 `type` 更加灵活，并且可以用于更多的场景。 `declare` **通常用于声明全局变量、函数、命名空间等外部类型**，而 `interface` 和 `type` 则更多用于**声明内部类型**。另外，`interface` 和 `type` 还支持继承、交叉类型等高级特性，使得类型声明更加丰富和灵活。

# 继承

继承是指一个类型继承另一个类型的属性和方法。在 TypeScript 中，可以使用 `extends` 关键字来继承一个接口或类的属性和方法。例如：

```ts
interface Animal {
  name: string;
  eat(): void;
}

interface Dog extends Animal {
  bark(): void;
}

class GoldenRetriever implements Dog {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
  eat(): void {
    console.log(`${this.name} is eating`);
  }
  bark(): void {
    console.log(`${this.name} is barking`);
  }
}
```

在上述代码中，`Dog` 接口继承了 `Animal` 接口的属性和方法，而 `GoldenRetriever` 类实现了 `Dog` 接口，并且具有 `name`、`eat()` 和 `bark()` 方法。

# 交叉类型

交叉类型是指将多个类型合并成一个类型。在 TypeScript 中，可以使用 `&` 符号来表示交叉类型。

## 案例一

```ts
interface Loggable {
  log(message: string): void;
}

interface Serializable {
  serialize(): string;
}

type LoggableAndSerializable = Loggable & Serializable;

class Person implements LoggableAndSerializable {
  log(message: string): void {
    console.log(message);
  }
  serialize(): string {
    return JSON.stringify(this);
  }
}
```

在上述代码中，`LoggableAndSerializable` 是一个交叉类型，它同时拥有 `Loggable` 和 `Serializable` 接口的属性和方法。 `Person` 类实现了 `LoggableAndSerializable` 接口，因此具有 `log()` 和 `serialize()` 方法。

## 案例二

```ts
interface A {
  a: string;
}

interface B {
  b: number;
}

type C = A & B;

const c: C = {
  a: "Hello",
  b: 42,
};
```

在上述代码中，`C` 类型是由 `A` 和 `B` 两个接口通过 `&` 符号合并而成的。 `C` 类型中包含了 `A` 和 `B` 接口中的所有属性和方法，因此可以用来表示既有 `a` 属性又有 `b` 属性的对象。

**继承**和**联合类型**和**交叉类型**都是 TypeScript 中非常强大的类型特性，可以用于将多个类型组合成一个更复杂的类型，提高代码的可读性和复用性。

# implements (关键字)

`implements` 是 TypeScript 中的一个关键字，用于类实现接口。当一个类实现了一个接口后，它需要实现接口中定义的所有属性和方法。如果没有实现接口中的所有属性和方法，TypeScript 编译器就会发出错误提示。

下面是一个例子：

```ts
interface Animal {
  name: string;
  eat(): void;
}

class Dog implements Animal {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
  eat(): void {
    console.log(`${this.name} is eating`);
  }
}

const dog = new Dog("Buddy");
dog.eat(); // 输出：Buddy is eating
```

在上述代码中，`Dog` 类需要实现 `Animal` 接口中定义的属性和方法，因此需要在 `Dog` 类中重新声明 `name` 和 `eat()` 方法。实际上，这里的 `name` 和 `eat()` 方法是 `Animal` 接口中定义的，但是在 `Dog` 类中需要重新声明一遍，并且实现它们的具体逻辑，才能让 `Dog` 类符合 `Animal` 接口的定义。

关于 `class Dog implements Animal`，它的作用是让 `Dog` 类实现 `Animal` 接口。通过这样的方式，TypeScript 就会检查 `Dog` 类是否实现了 `Animal` 接口中定义的所有属性和方法。如果 `Dog` 类没有实现 `Animal` 接口中的所有属性和方法，TypeScript 编译器就会发出错误提示。

`implements` 关键字可以帮助我们在编写代码时发现一些错误，例如忘记实现接口中的某个方法或属性等。它也可以提高代码的可读性和可维护性，使得代码更加规范和清晰。

在实际开发中，接口可以用来描述对象的结构和行为，而类可以用来实现接口中定义的行为。通过接口和类的结合使用，我们可以让代码更加规范和清晰，提高代码的可读性和可维护性。

# 类型保护

使用类型保护技术可以使代码更加健壮和可读，以下是一些常见的类型保护技术配合联合类型的示例：

## 1. typeof：使用 typeof **来检查变量的类型**

```ts
function printValue(value: string | number) {
  if (typeof value === 'string') {
    console.log(value.toUpperCase());
  } else {
    console.log(value.toFixed(2));
  }
}
```

## 2. instanceof：使用 instanceof 来检查变量是否是特定类的实例

```ts
class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

class Animal {
  type: string;
  constructor(type: string) {
    this.type = type;
  }
}

function printInfo(info: Person | Animal) {
  if (info instanceof Person) {
    console.log(`Person: ${info.name}`);
  } else {
    console.log(`Animal: ${info.type}`);
  }
}
```

## 3.   in：使用 in 来检查对象是否具有某个属性

```ts
interface Square {
  kind: 'square';
  size: number;
}

interface Rectangle {
  kind: 'rectangle';
  width: number;
  height: number;
}

type Shape = Square | Rectangle;

function getArea(shape: Shape) {
  if ('size' in shape) {
    return shape.size ** 2;
  } else {
    return shape.width * shape.height;
  }
}
```

## 4. 自定义类型保护函数：使用自定义类型保护函数来判断变量是否满足某些条件

```ts
interface Fish {
  type: 'fish';
  swim: () => void;
}

interface Bird {
  type: 'bird';
  fly: () => void;
}

type Animal = Fish | Bird;

function isFish(animal: Animal): animal is Fish {
  return (animal as Fish).type === 'fish';
}

function move(animal: Animal) {
  if (isFish(animal)) {
    animal.swim();
  } else {
    animal.fly();
  }
}
```



# tsconfig配置项

## noImplicitAny any类型检查

> Enable error reporting for expressions and declarations with an implied 'any' type.

在某些不存在类型批注的情况下， 当 TypeScript 无法推断出变量的类型时，它将回退到any类型

This can cause some errors to be missed, for example:

```typescript
function fn(s) {
  // No error?
  console.log(s.subtr(3));
}
fn(42);Try
```

Turning on `noImplicitAny` however TypeScript will issue an error whenever it would have inferred `any`:

```typescript
function fn(s) {Parameter 's' implicitly has an 'any' type.Parameter 's' implicitly has an 'any' type.
  console.log(s.subtr(3));
}
```

回想一下，在前面的某些例子中，TypeScript 没有为我们进行类型推断，这时候变量会采用最宽泛的类型：`any`。这并不是一件最糟糕的事情 —— 毕竟，使用 `any` 类型基本就和纯 JavaScript 一样了。

但是，使用 `any` 通常会和使用 TypeScript 的目的相违背。

你的程序使用越多的类型，那么在验证和工具上你的收益就越多，这意味着在编码的时候你会遇到越少的 bug。

启用 [noImplicitAny](https://www.typescriptlang.org/tsconfig#noImplicitAny) 配置项，在遇到被隐式推断为 `any` 类型的变量时就会抛出一个错误。

## strictNullChecks  null和undefined类型检查

默认情况下，`null` 和 `undefined` 可以被赋值给其它任意类型。

这会让你的编码更加容易，但世界上无数多的 bug 正是由于忘记处理 `null` 和 `undefined` 导致的 —— 有时候它甚至会带来[数十亿美元的损失](https://www.youtube.com/watch?v=ybrQvs4x0Ps)！

[strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks) 配置项让处理 `null` 和 `undefined` 的过程更加明显，让我们**不用**担心自己是否**忘记**处理 `null` 和 `undefined`。

- 设置为`false`时，`null`和`undefined`会被忽略检查，不会报错；
  ![image-20230308110008616](https://s2.loli.net/2023/03/08/dwNT89Qptg7hvXG.png)

- 设置为`true`时，会检测出可能会出现null和undefined的情况，并报错。
  ![image-20230308110023163](https://s2.loli.net/2023/03/08/kRnHC3fcZuyNMvT.png)

## noEmitOnError  

ts代码发生报错时，是否继续编译成js代码，默认为false，即报错也继续编译输出js文件。

设置为true，则发生报错时，停止编译。

# 函数类型表达式

描述函数的最简单方法是使用函数类型表达式。这些类型在语法上类似于箭头函数：

![image-20230505172908154](https://s2.loli.net/2023/05/05/DRVsqJMXcrhTWnm.png)

语法 `(a: string) => void` 表示“具有一个参数的函数，名为 `a` ，字符串类型，没有返回值”。就像函数声明一样，如果未指定参数类型，则隐式为 `any` 。

> 请注意，参数名称是必需的。函数类型 `(string) => void` 表示“具有名为 `string` 的参数的函数，类型为 `any` ”！

当然，我们可以使用类型别名来命名函数类型：

![image-20230505173023353](https://s2.loli.net/2023/05/05/dHQmOC7VNLKe98T.png)

# Call Signatures 调用签名

在 JavaScript 中，函数除了可调用之外，还可以具有属性。但是，函数类型表达式语法不允许声明属性。如果我们想描述一些可以用属性调用的东西，我们可以在对象类型中写一个调用签名：

> 实现函数上具有自定义属性，且有函数调用的参数类型声明

```ts
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}
 
function myFunc(someArg: number) {
  return someArg > 3;
}
myFunc.description = "default description";//在函数上挂载属性
 
doSomething(myFunc);
```

> 请注意，与函数类型表达式相比，语法略有不同 - 在参数列表和返回类型之间使用 `:` 而不是 `=>` 。
>

![image-20230505175053100](https://s2.loli.net/2023/05/05/LvSGakIcyTYD1jP.png)

# Construct Signatures 构造函数签名

JavaScript 函数也可以使用 `new` 运算符调用。TypeScript 将这些称为构造函数，因为它们通常会创建一个新对象。您可以通过在调用签名前面添加 `new` 关键字来编写构造签名：

```ts

// 普通函数
type SomeConstructor = (s: string) => void

function fn(ctor: SomeConstructor) {
  return ctor("hello")
}

fn(function A(s: string) {})

// 使用new的构造函数
type SomeConstructor2 = {
  new (s: string): Object
}
//或者 type SomeConstructor2 = new (s: string) => Object

function fn2(ctor: SomeConstructor2) {
  return new ctor("hello")
}

fn2(
  class A {
    constructor(s: string) {
      console.log(s)
    }
  }
)
```

有些对象，比如 JavaScript 的 `Date` 对象，可以用或不带 `new` 来调用。您可以任意组合同一类型的调用和构造签名：

```ts
interface CallOrConstruct {
  new (s: string): Date;
  (n?: number): number;
}
```

# Generic Functions 泛型函数 

通常编写一个函数，其中输入的类型与输出的类型相关，或者两个输入的类型以某种方式相关。让我们考虑一下返回数组第一个元素的函数：

```ts
function firstElement(arr: any[]) {
  return arr[0];
}
```

这个函数完成了它的工作，但不幸的是有返回类型 `any` 。如果函数返回数组元素的类型会更好。

在 TypeScript 中，泛型用于描述两个值之间的对应关系。我们通过在函数签名中声明一个类型参数来做到这一点：

```ts
function firstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0];
}
```

通过向此函数添加类型参数 `Type` 并在两个地方使用它，我们在函数的输入（数组）和输出（返回值）之间创建了一个链接。现在当我们调用它时，一个更具体的类型就出来了：

![image-20230508105136291](https://s2.loli.net/2023/05/08/kqHYCtlSFTsA7Gg.png)

## 推论

请注意，我们不必在此示例中指定 `Type` 。类型由 TypeScript 推断 - 自动选择。

我们也可以使用多个类型参数。例如， `map` 的独立版本如下所示：

```ts
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}
 
// Parameter 'n' is of type 'string'
// 'parsed' is of type 'number[]'
const parsed = map(["1", "2", "3"], (n) => parseInt(n));
```

请注意，在此示例中，TypeScript 可以根据函数表达式的返回值。

## 约束条件

我们已经编写了一些可以处理任何类型值的通用函数。有时我们想关联两个值，但只能对某个值的子集进行操作。在这种情况下，我们可以使用约束来限制类型参数可以接受的类型种类。

让我们编写一个返回两个值中较长值的函数。为此，我们需要一个 `length` 属性，它是一个数字。我们通过编写 `extends` 子句将类型参数限制为该类型：

```ts
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}
 
// longerArray is of type 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString is of type 'alice' | 'bob'
const longerString = longest("alice", "bob");
// Error! Numbers don't have a 'length' property
const notOK = longest(10, 100);
```

![image-20230508105902099](https://s2.loli.net/2023/05/08/x5PwOfIEi3WZDLQ.png)

在这个例子中有一些有趣的事情需要注意。我们允许 TypeScript 推断 `longest` 的返回类型。返回类型推断也适用于泛型函数。

因为我们将 `Type` 限制为 `{ length: number }` ，所以我们被允许访问 `a` 和 `b` 参数的 `.length` 属性。如果没有类型约束，我们将无法访问这些属性，因为这些值可能是没有长度属性的其他类型。

`longerArray` 和 `longerString` 的类型是根据参数推断出来的。请记住，泛型就是将两个或多个值与同一类型相关联！

![image-20230508111317370](https://s2.loli.net/2023/05/08/R8uJpkUhLy7eV6D.png)

![image-20230508111329346](https://s2.loli.net/2023/05/08/VtzgqupOZTLKGmn.png)

最后，如我们所愿，对 `longest(10, 100)` 的调用被拒绝，因为 `number` 类型没有 `.length` 属性。

# Working with Constrained Values 使用约束值

![image-20230508134320448](https://s2.loli.net/2023/05/08/qR8gQamzifK5WEp.png)

![image-20230508134336891](https://s2.loli.net/2023/05/08/rCNvoxYqGTHsp1J.png)

看起来这个函数没问题 - `Type` 被限制为 `{ length: number }` ，并且该函数返回 `Type` 或与该约束匹配的值。

这段代码的错误在于当传入的对象不符合约束条件时，返回的对象不是原类型，而是一个新的对象，这可能会导致类型不匹配的错误。应该使用类型断言来确保返回值的类型与传入的类型相同，例如：

```ts
function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number
): Type {
  if (obj.length >= minimum) {
    return obj;
  } else {
    return { length: minimum } as Type;
  }
}
```

## Specifying Type Arguments 指定类型参数

TypeScript 通常可以在泛型调用中推断出预期的类型参数，但并非总是如此。例如，假设您编写了一个函数来组合两个数组：

```js
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2)
}
```

通常用不匹配的数组调用此函数会出错：

![image-20230508140736854](https://s2.loli.net/2023/05/08/V9odchw4gnDlpX8.png)

但是，如果您打算这样做，您可以手动指定 `Type` ：

```js
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```

# 编写好的泛型函数的指南 Guidelines for Writing Good Generic Functions

编写泛型函数很有趣，而且很容易被类型参数冲昏头脑。拥有太多类型参数或在不需要它们的地方使用约束会降低推理的成功率，让函数的调用者感到沮丧。

## Push Type Parameters Down 下推类型参数

这里有两种编写看起来相似的函数的方法：

```ts
function firstElement1<Type>(arr: Type[]) {
  return arr[0];
}
 
function firstElement2<T extends any[]>(arr: T) {
  return arr[0]
}
 
// a: number (good)
const a = firstElement1([1, 2, 3]);
// b: any (bad)
const b = firstElement2([1, 2, 3]);
```

返回类型对比

![image-20230508170825949](https://s2.loli.net/2023/05/08/yXS1oqH4FdMZsnw.png)

![image-20230508170816261](https://s2.loli.net/2023/05/08/UFLvVdMrXqA9E8o.png)

乍一看，它们似乎相同，但 `firstElement1` 是编写此函数的更好方法。它的推断返回类型是 `Type` ，但 `firstElement2` 的推断返回类型是 `any` ，因为 TypeScript 必须使用约束类型解析 `arr[0]` 表达式，而不是在调用期间“等待”解析元素。

> **Rule**: When possible, use the type parameter itself rather than constraining it.

这句话的意思是在 TypeScript 中，尽可能使用类型参数本身而不是对其进行约束。这是因为类型参数本身已经表示了所需要的类型信息，不需要额外的约束来限制它的类型。这种做法可以使代码更加简洁和清晰。例如，使用类型参数 T 而不是对其进行约束，可以让 T 表示任何类型，而不仅限于特定的类型。

当使用类型参数本身时，代码可能如下所示：

```ts
function identity<T>(arg: T): T {
  return arg;
}

let result = identity("hello");
```

在这个示例中，我们定义了一个泛型函数 `identity`，它接受一个类型参数 `T`，并返回一个类型为 `T` 的值。在函数体中我们只是简单地返回了传入的参数 `arg`。在调用 `identity` 函数时，我们将字符串 `"hello"` 作为参数传入，并将返回值赋给变量 `result`。

如果使用类型约束，代码可能如下所示：

```ts
function identity<T extends string>(arg: T): T {
  return arg;
}

let result = identity("hello");
```

在这个示例中，我们使用了类型约束 `T extends string`，将类型参数 `T` 约束为只能是 `string` 类型或其子类型。这样，调用 `identity` 函数时，只能传入字符串类型的参数。但是，这种方式可能会限制了 `identity` 函数的灵活性，因为它只能接受特定类型的参数。

## Use Fewer Type Parameters 使用更少的类型参数

这是另一对类似的函数：

```ts
function filter1<Type>(arr: Type[], func: (arg: Type) => boolean): Type[] {
  return arr.filter(func);
}
 
function filter2<Type, Func extends (arg: Type) => boolean>(
  arr: Type[],
  func: Func
): Type[] {
  return arr.filter(func);
}
```

在第二个函数中，我们创建了一个不关联两个值的类型参数 `Func` 。这始终是一个危险信号，因为这意味着想要指定类型参数的调用者必须无缘无故地手动指定一个额外的类型参数。 `Func` 除了让函数更难阅读和推理之外什么也没做！

```ts
let aa = filter1<number>([1,2,3],(item) => item > 2)
let bb = filter2<number,(arg: number) => boolean>([1,2,3],(item) => item > 2)
```

![image-20230508185758433](https://s2.loli.net/2023/05/08/BTFpy2t1QDLCrus.png)

这里就要无缘无故要多写一个额外的类型参数。

> **Rule**: Always use as few type parameters as possible
> 规则：始终使用尽可能少的类型参数

## Type Parameters Should Appear Twice 类型参数应该出现两次

有时我们会忘记一个函数可能不需要是泛型的：

```ts
function greet<Str extends string>(s: Str) {
  console.log("Hello, " + s);
}
 
greet("world");
```

我们可以很容易地编写一个更简单的版本：

```ts
function greet(s: string) {
  console.log("Hello, " + s);
}
```

请记住，类型参数用于关联多个值的类型。如果一个类型参数在函数签名中只使用一次，那么它没有任何关联。

> **Rule**: If a type parameter only appears in one location, strongly reconsider if you actually need it
> 规则：如果一个类型参数只出现在一个位置，强烈重新考虑你是否真的需要它



# Optional Parameters 可选参数

JavaScript 中的函数通常采用可变数量的参数。例如， `number` 的 `toFixed` 方法采用可选的数字计数：

```ts
function f(n: number) {
  console.log(n.toFixed()); // 0 arguments
  console.log(n.toFixed(3)); // 1 argument
}
```

我们可以通过使用 `?` 将参数标记为可选来在 TypeScript 中对其进行编写：

```ts
function f(n?:number) {
    // console.log(n.toFixed());
}
f()
f(10)
```

虽然参数被指定为 `number` 类型，但 `x` 参数实际上将具有 `number | undefined` 类型，因为JavaScript 中未指定的参数获得值 `undefined` 。

你可以设置一个默认值：

> 现在在 `f` 的主体中， `x` 将具有类型 `number` ，因为任何 `undefined` 参数都将被替换为 `10` 。

```ts
function f(n = 10) {
    // console.log(n.toFixed());
}
```

请注意，当参数是可选的时，调用者始终可以传递 `undefined` ，因为这只是模拟了一个“缺失”参数：

```ts
declare function f2(n?: number): void;
f2();
f2(10);
f2(undefined)
```

## Optional Parameters in Callbacks 回调中的可选参数

一旦了解了可选参数和函数类型表达式，在编写调用回调的函数时很容易犯以下错误：

![image-20230516163621619](https://s2.loli.net/2023/05/16/SmBKCX9GgrvAeJ5.png)

> **Rule**: When writing a function type for a callback, *never* write an optional parameter unless you intend to *call* the function without passing that argument
>
> 规则：为回调编写函数类型时，切勿编写可选参数，除非您打算在不传递该参数的情况下调用该函数

# Function Overloads 函数重载 

一些函数可以在不同的参数数量和类型的情况下调用。例如，您可以编写一个函数来生成 `Date` ，它采用时间戳（一个参数）或月/日/年规范（三个参数）。

在 TypeScript 中，我们可以通过编写重载签名*overload signatures*来指定一个可以以不同方式调用的函数。为此，编写一些函数签名（通常是两个或更多），然后是函数体：

![image-20230516170334398](https://s2.loli.net/2023/05/16/EQXd4YBPjDVASmn.png)

在这个例子中，我们写了两个重载：一个接受一个参数，另一个接受三个参数。前两个签名称为重载签名。

然后，我们编写了一个具有兼容签名的函数实现。函数有一个实现签名，但是这个签名不能被直接调用。即使我们在必需的参数之后写了一个带有两个可选参数的函数，它也不能用两个参数调用！

## Overload Signatures and the Implementation Signature 重载签名和实现签名

这是一个常见的混淆来源。通常人们会写这样的代码并且不明白为什么会出错：

```ts
function fn(x: string): void;
function fn() {
  // ...
}
// Expected to be able to call with zero arguments
fn();
Expected 1 arguments, but got 0.
```

同样，用于编写函数体的签名不能从外部“看到”。

> 实现的签名从外部是不可见的。编写重载函数时，您应该始终在函数的实现之上有两个或更多签名

实现签名还必须与重载签名兼容。例如，这些函数有错误，因为实现签名没有以正确的方式匹配重载：

> 在这个例子中，如果传入的参数是字符串类型，则返回值类型为字符串类型；如果传入的参数是数字类型，则返回值类型为布尔类型。但是，无论传入的参数类型是什么，函数体中始终返回的是字符串 "oops"。这可能会导致函数的行为与预期不符,

```ts
function fn(x: string): string;
// Return type isn't right
function fn(x: number): boolean;
// This overload signature is not compatible with its implementation signature.
function fn(x: string | number) {
  return "oops";
}
```

> 这个例子中，函数实现的参数类型和上面的函数前面不一致。

```ts
function fn(x: boolean): void;
// Argument type isn't right
function fn(x: string): void;
// This overload signature is not compatible with its implementation signature.
function fn(x: boolean) {}
```

正确的实现：

> 在函数体中，我们使用 `typeof` 运算符判断参数的类型，如果是字符串类型，则将其转换为大写字母并返回；如果是数字类型，则判断其是否大于 0 并返回相应的布尔值。

```ts
function fn2(x: string): string;
function fn2(x: number): boolean;
function fn2(x: string | number) {
  if (typeof x === 'string') {
    return x.toUpperCase();
  } else {
    return x > 0;
  }
}
```

## Writing Good Overloads 编写好的重载

与泛型一样，在使用函数重载时也应遵循一些准则。遵循这些原则会让你的函数更容易调用，更容易理解，更容易实现。

让我们考虑一个返回字符串或数组长度的函数：

```ts
function len(s: string): number;
function len(arr: any[]): number;
function len(x: any) {
  return x.length;
}
```

这个功能很好；我们可以用字符串或数组调用它。但是，我们不能使用可能是字符串或数组的值来调用它，因为 TypeScript 只能将函数调用解析为单个重载：

```ts
function len(s: string): number;
function len(s: any[]): number;
function len(x: any) {
    return x.length;
}

len(""); // OK
len([0]); // OK
len(Math.random() > 0.5 ? "hello" : [0]);
```

![image-20230516173705739](https://s2.loli.net/2023/05/16/cXwxo5K4s1pJ8Gu.png)

因为两个重载具有相同的参数计数和相同的返回类型，我们可以改为编写函数的非重载版本：

```ts
function len(x: any[] | string) {
  return x.length;
}
```

这好多了！调用者可以使用任何一种值来调用它，作为额外的好处，我们不必找出正确的实现签名。

> Always prefer parameters with union types instead of overloads when possible
> 在可能的情况下，始终优先使用具有联合类型的参数而不是重载

### Declaring `this` in a Function 函数中的this

TypeScript 将通过代码流分析推断函数中 `this` 的含义，例如：

```ts
const user = {
  id: 123,
 
  admin: false,
  becomeAdmin: function () {
    this.admin = true;
  },
};
```

TypeScript 理解函数 `user.becomeAdmin` 有一个对应的 `this` ，它是外部对象 `user` 。 `this` ，在很多情况下就足够了，但是在很多情况下，您需要更多地控制 `this` 代表的对象。 JavaScript 规范声明你不能有一个名为 `this` 的参数，因此 TypeScript 使用该语法空间让你在函数体中声明 `this` 的类型。

![image-20230516183110043](https://s2.loli.net/2023/05/16/LGeiMoQRIm58zkO.png)

![image-20230516183149423](https://s2.loli.net/2023/05/16/RuQ3NbLKd1Oer6C.png)

这种模式在回调式 API 中很常见，在这种情况下，另一个对象通常会控制何时调用您的函数。请注意，您需要使用 `function` 而不是箭头函数来获得此行为：

![image-20230516183207866](https://s2.loli.net/2023/05/16/JInel4DFWYLhvc9.png)

# Other Types to Know About 其他需要知道的类型

在使用函数类型时，您需要识别一些经常出现的其他类型。与所有类型一样，您可以在任何地方使用它们，但它们在函数上下文中尤为重要。

## void

`void` 表示不返回值的函数的返回值。是一种推断类型，只要函数没有任何 `return` 语句，或者不从这些 return 语句返回任何显式值。

```ts
// The inferred return type is void
function noop() {
  return;
}
```

![image-20230516183551751](https://s2.loli.net/2023/05/16/YeMSiC3BlraUjOp.png)

在 JavaScript 中，不返回任何值的函数将隐式返回值 `undefined` 。但是， `void` 和 `undefined` 在 TypeScript 中不是一回事。

> `void` is not the same as `undefined`.
> `void` 与 `undefined` 不同

- `void` 表示没有返回值的函数或表达式的类型。
- `undefined` 表示一个变量或表达式的值为未定义或不存在。

`void` 和 `undefined` 是不同的类型，但是它们之间存在一定的关系。如果一个函数的返回值类型为 `void`，则该函数的返回值为 `undefined`。

## object

特殊类型 `object` 指的是任何不是原始类型的值（ `string` 、 `number` 、 `bigint` 、 `boolean` 、 `symbol` 、 `null` 或 `undefined` ）。这不同于空对象类型 `{ }` ，也不同于全局类型 `Object` 。您很可能永远不会使用 `Object` 。

> `object` is not `Object`. **Always** use `object`!
> `object` 不是 `Object` 。始终使用 `object` ！

请注意，在 JavaScript 中，函数值是对象：它们具有属性，在其原型链中具有 `Object.prototype` ，是 `instanceof Object` ，您可以对它们调用 `Object.keys` ，等等。因此，函数类型在 TypeScript 中被视为 `object` 。

## unknown

`unknown` 类型表示任何值。这类似于 `any` 类型，但更安全，因为使用 `unknown` 值做任何事情都是不合法的：

```ts
function f1(a: any) {
  a.b(); // OK
}
function f2(a: unknown) {
  a.b();
// 'a' is of type 'unknown'.
}
```

这在描述函数类型时很有用，因为您可以描述接受任何值的函数，而无需在函数主体中包含 `any` 值。

相反，您可以描述一个返回未知类型值的函数：

```ts
function safeParse(s: string): unknown {
  return JSON.parse(s);
}
 
// Need to be careful with 'obj'!
const obj = safeParse(someRandomString);
```

## never

有些函数从不返回值：

```ts
function fail(msg: string): never {
  throw new Error(msg);
}
```

`never` 类型表示从未观察到的值。在返回类型中，这意味着函数抛出异常或终止程序的执行。

当 TypeScript 确定联合中没有任何内容时，也会出现 `never` 。

![image-20230517104510687](https://s2.loli.net/2023/05/17/rBGoQLPJwkaq74e.png)

## Function

全局类型 `Function` 描述了诸如 `bind` 、 `call` 、 `apply` 和其他出现在 JavaScript 中所有函数值上的属性。它还具有 `Function` 类型的值总是可以被调用的特殊属性；这些调用返回 `any` ：

```ts
function doSomething(f: Function) {
  return f(1, 2, 3);
}
```

这是一个无类型的函数调用，通常最好避免，因为 `any` 返回类型不安全。

如果您需要接受任意函数但不打算调用它， `() => void` 类型通常更安全。

# Rest Parameters and Arguments 剩余形参和参数

## Rest Parameters 剩余形参

除了使用可选参数或重载来制作可以接受各种固定参数计数的函数外，我们还可以使用剩余参数定义接受无限数量参数的函数。

其余参数出现在所有其他参数之后，并使用 `...` 语法：

```ts
function multiply(n: number, ...m: number[]) {
  return m.map((x) => n * x);
}
// 'a' gets value [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4);
```

在 TypeScript 中，这些参数上的类型注释隐式地是 `any[]` 而不是 `any` ，并且给定的任何类型注释都必须是 `Array<T>` 或 `T[]` 形式，或者是元组类型（我们将在后面学习） ).

## Rest Arguments 剩余参数

相反，我们可以使用扩展语法从数组中提供可变数量的参数。例如，数组的 `push` 方法可以接受任意数量的参数：

```ts
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
arr1.push(...arr2);
```

请注意，通常，TypeScript 不会假定数组是不可变的。这可能会导致一些令人惊讶的行为：

```ts
// Inferred type is number[] -- "an array with zero or more numbers",
// not specifically two numbers
const args = [8, 5];
const angle = Math.atan2(...args);
//A spread argument must either have a tuple type or be passed to a rest parameter.
```

![image-20230517113114132](https://s2.loli.net/2023/05/17/9asOcTedXhq4Mn2.png)

这种情况的最佳解决方案在一定程度上取决于您的代码，但通常 `const` 上下文是最直接的解决方案：

```ts
// Inferred as 2-length tuple
const args = [8, 5] as const;
// OK
const angle = Math.atan2(...args);
```

![image-20230517113214226](https://s2.loli.net/2023/05/17/GVR7ODWxbuyqT8t.png)

# Parameter Destructuring 参数解构

您可以使用参数解构来方便地将作为参数提供的对象解包到函数体中的一个或多个局部变量中。在 JavaScript 中，它看起来像这样：

```ts
function sum({ a, b, c }) {
  console.log(a + b + c);
}
sum({ a: 10, b: 3, c: 9 });
```

对象的类型注释在解构语法之后：

```ts
function sum({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c);
}
```

这看起来有点冗长，但您也可以在此处使用命名类型：

```ts
// Same as prior example
type ABC = { a: number; b: number; c: number };
function sum({ a, b, c }: ABC) {
  console.log(a + b + c);
}
```

# Assignability of Functions 功能的可分配性

### Return type`void`

函数的 `void` 返回类型可能会产生一些异常但符合预期的行为。

返回类型为 `void` 的上下文类型不会强制函数不返回某些内容。另一种说法是具有 `void` 返回类型 ( `type vf = () => void` ) 的上下文函数类型，在实现时**可以返回任何其他值**，但它将被忽略。

因此， `() => void` 类型的以下实现是有效的：

```ts
type voidFunc = () => void;
 
const f1: voidFunc = () => {
  return true;
};
 
const f2: voidFunc = () => true;
 
const f3: voidFunc = function () {
  return true;
};
```

而当其中一个函数的返回值赋值给另一个变量时，它会保留 `void` 的类型：

```ts
const v1 = f1();
 
const v2 = f2();
 
const v3 = f3();
```

此行为的存在使得以下代码有效，即使 `Array.prototype.push` 返回一个数字并且 `Array.prototype.forEach` 方法需要一个返回类型为 `void` 的函数。

```ts
const src = [1, 2, 3];
const dst = [0];
 
src.forEach((el) => dst.push(el));
```

还有一种需要注意的特殊情况，当文字函数定义具有 `void` 返回类型时，该函数不得返回任何内容。

```ts
function f2(): void {
  // @ts-expect-error
  return true;
}
 
const f3 = function (): void {
  // @ts-expect-error
  return true;
};
```

### @ts-expect-error

`@ts-expect-error` 是 TypeScript 中的一个特殊注释，用于告诉 TypeScript 编译器在该行代码上忽略类型检查错误。通常情况下，TypeScript 编译器会在代码中发现类型检查错误时报错，以帮助开发者尽早发现和修复问题。但是，在某些情况下，我们可能需要暂时忽略某些类型检查错误，以便进行一些实验性的代码编写或快速的原型开发。

使用 `@ts-expect-error` 注释可以告诉 TypeScript 编译器在该行代码上忽略类型检查错误。

```ts
const x: string = 123; // Type 'number' is not assignable to type 'string'.
// @ts-expect-error
const y: string = 123; // No error is reported by TypeScript compiler.
```

