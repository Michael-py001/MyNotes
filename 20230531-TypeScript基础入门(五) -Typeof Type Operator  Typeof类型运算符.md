# 20230531-Typeof Type Operator | `Typeof` 类型运算符

JavaScript已经有一个 `typeof` 运算符，您可以在表达式上下文中使用：

```ts
// Prints "string"
console.log(typeof "Hello world");
```

TypeScript 添加了一个 `typeof` 运算符，您可以在类型上下文中使用它来引用变量或属性的类型：

![image-20230531155326493](https://s2.loli.net/2023/05/31/P1ASm3NdeKJncIG.png)

这对于基本类型来说并不是很有用，但与其他类型运算符结合使用，您可以使用 `typeof` 方便地表达许多模式。例如，让我们首先看一下预定义类型 `ReturnType<T>` 。它接受一个函数类型并生成其返回类型：

![image-20230531155649437](https://s2.loli.net/2023/05/31/ojmJaP6fSphuczq.png)

如果我们尝试在函数名称上使用 `ReturnType` ，我们会看到一个有指导意义的错误：

![image-20230531155919801](https://s2.loli.net/2023/05/31/CHDBfgFIE5lrVvX.png)

记住，值和类型不是同一回事。要引用值 `f` 所具有的类型，我们使用 `typeof` ：

![image-20230531160018945](https://s2.loli.net/2023/05/31/ohfV5Zs73vaXew8.png)

## Limitations  局限性

TypeScript 有意限制了您可以在 `typeof` 上使用的表达式类型。

具体来说，只有在标识符（即变量名）或其属性上使用 `typeof` 才是合法的。这有助于避免编写您认为正在执行但实际上并未执行的代码的混淆陷阱：

![image-20230531160845638](https://s2.loli.net/2023/05/31/lhTBg2fmjcJ7aZK.png)