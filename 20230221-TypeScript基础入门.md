# 20230221-TypeScript基础入门

## 安装



# TS与JS中的类型

JavaScript 中已经有一些基本类型可用：`boolean`、 `bigint`、 `null`、`number`、 `string`、 `symbol `和 `undefined`，引用数据类型：`object`，一共8种， 它们都可以在接口中使用。TypeScript 将此列表扩展为更多的内容，例如:

- `any` （允许任何类型）
- [`unknown`](https://www.typescriptlang.org/play#example/unknown-and-never) （确保使用此类型的人声明类型是什么）
-  [`never`](https://www.typescriptlang.org/play#example/unknown-and-never) （这种类型不可能发生）
-  `void` （返回 `undefined` 或没有返回值的函数）。

所以TS中的类型一共有 8+4 = 12种。

构建类型有两种语法： [接口和类型](https://www.typescriptlang.org/play/?e=83#example/types-vs-interfaces)。 你应该更喜欢 `interface`。当需要特定功能时使用 `type` 。

# 定义类型

要创建具有推断类型的对象，该类型包括 `name: string` 和 `id: number`，你可以这么写：

```typescript
const user = {
  name: "Hayes",
  id: 0,
};
```

![image-20230222154813566](https://s2.loli.net/2023/02/22/5zSvFLiaynOYlkM.png)

## 用interface定义类型 

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

## 用type定义类型

```typescript
type BirdType = {
  wings: 2;
};

const bird1: BirdType = { wings: 2 };
```

# type 和interface 定义类型对比

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

## 联合类型

使用联合，可以声明类型可以是许多类型中的一种。例如，可以将 `boolean` 类型描述为 `true` 或 `false` ：

```ts
type MyBool = true | false;
```

注意：_如果将鼠标悬停在上面的 `MyBool` 上，您将看到它被归类为 `boolean`。这是结构化类型系统的一个属性。下面有更加详细的信息。

![image-20230223115905248](https://s2.loli.net/2023/02/23/GVhcmNdUkxCEfQ4.png)

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

# 断言

从 TypeScript 1.6 版本开始，TypeScript 提供了类型断言（Type Assertion）机制，可以用来手动指定变量的类型，以适应开发者的特定需求。

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

## strictNullChecks  null类型检查

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
