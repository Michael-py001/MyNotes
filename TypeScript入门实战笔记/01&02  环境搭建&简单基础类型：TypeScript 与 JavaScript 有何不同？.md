# 20230510-TypeScript入门实战笔记

# 01  | TS开发环境搭建

基于 VS Code 完善 TypeScript 开发、转译环境。

使用“Ctrl + `”快捷键打开 VS Code 内置命令行工具，输入如下所示代码:

```
npm i -g typescript
```

TypeScript 安装完成后，输入以下命令查看版本：

```
tsc -v
```

我们也可以通过安装在 Terminal 命令行中直接支持运行 TypeScript 代码（Node.js 侧代码）的 ts-node 来获得较好的开发体验。

```
npm i -g ts-node
```

使用ts-node运行ts代码：

```
ts-node HelloWorld.ts
```

# 02 | 简单基础类型：TypeScript 与 JavaScript 有何不同？

## TypeScript 简介

TypeScript 其实就是类型化的 JavaScript，它不仅支持 JavaScript 的所有特性，还在 JavaScript 的基础上添加了静态类型注解扩展。

比如 JavaScript 中虽然提供了原始数据类型 string、number，但是它无法检测我们是不是按照约定的类型对变量赋值，而 TypeScript 会对赋值及其他所有操作默认做静态类型检测。

因此，从某种意义上来说，**TypeScript 其实就是 JavaScript 的超集**，如下图所示：

![image-20230510113304348](https://s2.loli.net/2023/05/10/2k85foCgpBGPJmW.png)

## 基本语法

TypeScript 语法与 JavaScript 语法的区别在于，我们可以在 TypeScript 中显式声明变量`num`仅仅是数字类型，也就是说只需在变量`num`后添加`: number`类型注解即可，如下代码所示：

```ts
let num: number = 1;
```

**特殊说明：**`number`表示数字类型，`:`用**来分割变量和类型的分隔符。**

同理，我们也可以把`:`后的`number`换成其他的类型（比如 JavaScript 原始类型：number、string、boolean、null、undefined、symbol 等），此时，num 变量也就拥有了 TypeScript 同名的原始类型定义。

关于 JavaScript 原始数据类型到 TypeScript 类型的映射关系如下表所示：

![image-20230510113848225](https://s2.loli.net/2023/05/10/LncAeOSxXqHJUYR.png)

### 原始类型

在 JavaScript 中，**原始类型指的是非对象且没有方法的数据类型**，它包括 string、number、bigint、boolean、undefined 和 symbol 这六种 **（null 是一个伪原始类型，它在 JavaScript 中实际上是一个对象，且所有的结构化类型都是通过 null 原型链派生而来）**。

在 JavaScript 语言中，原始类型值是最底层的实现，对应到 TypeScript 中同样也是最底层的类型。

### 字符串

```ts
let firstname: string = 'Captain'; // 字符串字面量
let familyname: string = String('S'); // 显式类型转换
let fullname: string = `my name is ${firstname}.${familyname}`; // 模板字符串
```

### 数字

```ts
/** 十进制整数 */
let integer: number = 6;
/** 十进制整数 */
let integer2: number = Number(42);
```

- bigint 

  ```ts
  let big: bigint =  100n;
  ```

  **请注意：虽然**`number`和`bigint`都表示数字，但是这两个类型不兼容。

### 布尔值

```ts
/** TypeScript 真香 为 真 */
let TypeScriptIsGreat: boolean = true;
 /** TypeScript 太糟糕了 为 否 */
let TypeScriptIsBad: boolean = false;
```

### Symbol

```ts
let sym1: symbol = Symbol();
let sym2: symbol = Symbol('42');
```

**当然，TypeScript 还包含 Number、String、Boolean、Symbol 等类型（注意区分大小写）**

> **特殊说明：请你千万别将它们和小写格式对应的 number、string、boolean、symbol 进行等价**。不信的话，你可以思考并验证如下所示的示例。

![image-20230510114908242](https://s2.loli.net/2023/05/10/VPLrK7xqEjT34gs.png)

![image-20230510114854324](https://s2.loli.net/2023/05/10/dVf4x9D3sb2CXPH.png)

实际上，我们压根使用不到 Number、String、Boolean、Symbol 类型，因为它们并没有什么特殊的用途。这就像我们不必使用 JavaScript Number、String、Boolean 等构造函数 new 一个相应的实例一样。

介绍完这几种原始类型后，你可能会心生疑问：缺省类型注解的有无似乎没有什么明显的作用，就像如下所示的示例一样：

```ts
{
  let mustBeNum = 1;
}
{
  let mustBeNum: number = 1;
}
```

其实，以上这两种写法在 TypeScript 中是等价的，这得益于基于上下文的类型推导(后面会讲到)。

下面，我们对上面的示例稍做一下修改，如下代码所示：

```ts
{
  let mustBeNum = 'badString';
}
{
  let mustBeNum: number = 'badString';
}
```

![image-20230510120027581](https://s2.loli.net/2023/05/10/ltd7uZLORWNcpSi.png)

以上就是类型注解作用的直观体现。

## 静态类型检测

在编译时期，静态类型的编程语言即可准确地发现类型错误，这就是静态类型检测的优势。

在编译（转译）时期，TypeScript 编译器将通过对比检测变量接收值的类型与我们显示注解的类型，从而检测类型是否存在错误。如果两个类型完全一致，显示检测通过；如果两个类型不一致，它就会抛出一个编译期错误，告知我们编码错误，具体示例如下代码所示：

```ts
const trueNum: number = 42;
const fakeNum: number = "42"; // ts(2322) Type 'string' is not assignable to type 'number'.
```

### 小结

这一讲通过与 JavaScript 的基础类型进行对比，我们得知：TypeScript 其实就是添加了类型注解的 JavaScript，它并没有任何颠覆性的变动。因此，学习并掌握 TypeScript 一定会是一件极其容易的事情。