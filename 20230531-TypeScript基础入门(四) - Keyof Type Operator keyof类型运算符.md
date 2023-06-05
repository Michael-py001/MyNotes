# Keyof Type Operator `keyof`类型运算符

`keyof` 运算符接受一个对象类型并生成其键的字符串或数字文字联合。以下类型 `P` 与 `type P = "x" | "y"` 相同：

```ts
type Point = { x: number; y: number };
type P = keyof Point;
// `type P = keyof Point`
```

如果类型具有字符串或数字**索引签名**，keyof 将返回这些类型：

![image-20230531152351313](https://s2.loli.net/2023/05/31/IilM9pvasGRo8DE.png)

请注意，在此示例中， `M` 类型是 `string | number`，这是因为 JavaScript 对象键始终被强制转换为字符串，因此 `obj[0]` 始终与 `obj["0"]` 相同。

当与映射类型结合使用时， `keyof` 类型变得特别有用，我们稍后将学习更多相关知识。