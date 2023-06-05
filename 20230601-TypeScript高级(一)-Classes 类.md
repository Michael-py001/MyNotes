# 20230601-TypeScript高级(一)-Classes 类.md

TypeScript 完全支持 ES2015 中引入的 `class` 关键字。

与其他 JavaScript 语言功能一样，TypeScript 添加了类型注释和其他语法，以允许您表达类和其他类型的关系。

## Class Members

这是最基本的类 - 一个空的类：

```ts
class Point {}
```

这个类还不是很有用，所以让我们开始添加一些成员。

### Fields

字段声明在类上创建公共可写属性：

```ts
class Point {
  x: number;
  y: number;
}
 
const pt = new Point();
pt.x = 0;
pt.y = 0;
```

与其他位置一样，类型注释是可选的，但如果未指定，则将为隐式 `any` 。

字段也可以具有初始值设定项;这些将在类实例化时自动运行：

```ts
class Point {
  x = 0;
  y = 0;
}
 
const pt = new Point();
// Prints 0, 0
console.log(`${pt.x}, ${pt.y}`);
```

就像 `const` 、 `let` 和 `var` 一样，类属性的初始值设定项将用于推断其类型：

![image-20230601155037468](https://s2.loli.net/2023/06/01/ByxPGQ5dUjwrcF1.png)

#### `strictPropertyInitialization`

[`strictPropertyInitialization`](https://www.typescriptlang.org/tsconfig#strictPropertyInitialization) 设置控制是否需要在构造函数中初始化类字段。

![image-20230601155301771](https://s2.loli.net/2023/06/01/cspKeyrkN5jYziH.png)

```ts
class GoodGreeter {
  name: string;
 
  constructor() {
    this.name = "hello";
  }
}
```

请注意，该字段需要在构造函数本身中初始化。TypeScript 不会分析从构造函数调用的方法来检测初始化，因为派生类可能会重写这些方法并且无法初始化成员。

如果您打算通过构造函数以外的方式明确初始化字段（例如，可能外部库正在为您填充类的一部分），则可以使用定义赋值断言运算符 `!` ：

```ts
class OKGreeter {
  // Not initialized, but no error
  name!: string;
}
```

## readonly

字段可以以 `readonly` 修饰符为前缀。这可以防止赋值到构造函数外部的字段。

![image-20230601160200422](https://s2.loli.net/2023/06/01/8LT1zRto6GUckdm.png)

### Constructors

类构造函数与函数非常相似。您可以使用类型注释、默认值和重载添加参数：

```ts
class Point {
  x: number;
  y: number;
 
  // Normal signature with defaults
  constructor(x = 0, y = 0) {
    this.x = x;
    this.y = y;
  }
}
```

```ts
class Point {
  // Overloads
  constructor(x: number, y: string);
  constructor(s: string);
  constructor(xs: any, y?: any) {
    // TBD
  }
}
```

类构造函数签名和函数签名之间只有一些区别：

- 构造函数不能有类型参数 - 这些参数属于外部类声明，我们将在后面学习

- 构造函数不能有返回类型注释 - 类实例类型始终是返回的内容

#### Super Calls

就像在 JavaScript 中一样，如果你有一个基类，你需要在使用任何 `this.` 成员之前在构造函数主体中调用 `super();` ：

![image-20230601161554703](https://s2.loli.net/2023/06/01/lSdAz86Es4gp7Mj.png)

忘记调用 `super` 在 JavaScript 中很容易犯错误，但 TypeScript 会告诉你什么时候有必要。

### Methods

类上的函数属性称为方法。方法可以使用所有相同类型的注释作为函数和构造函数：

```ts
class Point {
  x = 10;
  y = 10;
 
  scale(n: number): void {
    this.x *= n;
    this.y *= n;
  }
}
```

除了标准类型注释之外，TypeScript 不会向方法添加任何其他新内容。

请注意，在方法主体中，仍然必须通过 `this.` 访问字段和其他方法。方法主体中的非限定名称将始终引用封闭作用域中的某些内容：

![image-20230601161919789](https://s2.loli.net/2023/06/01/XiLyElnAp8PhNYS.png)

### Getters / Setters

```ts
class C {
  _length = 0;
  get length() { // Getters 
    return this._length;
  }
  set length(value) { // Setters
    this._length = value;
  }
}
```

> 请注意，没有额外逻辑的字段支持的 get/set 对在 JavaScript 中很少有用。如果在获取/设置操作期间不需要添加其他逻辑，则可以公开公共字段。

TypeScript 对访问者有一些特殊的推理规则：

- 如果存在 `get` 但没有 `set` ，则属性自动为 `readonly`

- 如果未指定 setter 参数的类型，则从 getter 的返回类型推断
- Getters and setters 必须具有相同的成员可见性

从 TypeScript 4.3 开始，可以使用不同类型的访问器来获取和设置。

```ts
class Thing {
  _size = 0;
 
  get size(): number { //number类型
    return this._size;
  }
 
  set size(value: string | number | boolean) {
    let num = Number(value);
 
    // Don't allow NaN, Infinity, etc
 
    if (!Number.isFinite(num)) {
      this._size = 0;
      return;
    }
 
    this._size = num;
  }
}
```

### Index Signatures 索引签名

类可以声明索引签名;这些工作方式与其他对象类型的索引签名相同：

```ts
class MyClass {
  [s: string]: boolean | ((s: string) => boolean);
 
  check(s: string) {
    return this[s] as boolean;
  }
}
```

由于索引签名类型还需要捕获方法的类型，因此不容易有效地使用这些类型。通常，最好将索引数据存储在另一个位置，而不是存储在类实例本身上。

## implements 实现接口

> **一句话解释**：`class A implements B`  ，A这个**类**必须要实现B这个**接口**的所有属性。

在 TypeScript 中，`implements` 关键字用于实现接口。具体来说，当一个类实现了一个接口时，它必须实现接口中定义的所有属性和方法。

例如，假设我们有一个名为 `Person` 的接口，它定义了一个名为 `name` 的属性和一个名为 `sayHello` 的方法：

```ts
interface Person {
  name: string;
  sayHello(): void;
}
```

现在我们想创建一个类 `Student`，它实现了 `Person` 接口，可以这样写：

```ts
class Student implements Person {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  sayHello() {
    console.log(`Hello, my name is ${this.name}.`);
  }
}
```

在上面的例子中，我们使用 `implements` 关键字将 `Student` 类与 `Person` 接口关联起来，表示 `Student` 类实现了 `Person` 接口。因此，`Student` 类必须实现 `Person` 接口中定义的所有属性和方法，包括 `name` 属性和 `sayHello` 方法。

需要注意的是，当一个类实现了一个接口时，它可以实现多个接口，例如：

```ts
class Student implements Person, Teacher {
  // ...
}
```

在上面的例子中，`Student` 类实现了 `Person` 接口和 `Teacher` 接口。这样，`Student` 类就必须实现 `Person` 接口和 `Teacher` 接口中定义的所有属性和方法。

> 理解 implements 子句只是检查类是否可以被视为接口类型是很重要的。它不会改变类或其方法的类型。一个常见的错误是假设 implements 子句会改变类的类型，但实际上它并不会这样做！



```ts
interface Checkable {
  check(name: string): boolean;
}

class NameChecker implements Checkable {
  check(s) {
    return s.toLowerCase() === "ok"
  }
}
```

![image-20230602183807899](https://s2.loli.net/2023/06/02/FeMu7tXnhSl6bDH.png)

在这个例子中，我们可能期望 `s` 的类型会受到 `check` 函数的 `name: string` 参数的影响。但实际上并不会 - `implements` 子句不会改变类体的检查方式或其类型推断。

同样，使用可选属性实现接口也不会创建该属性：