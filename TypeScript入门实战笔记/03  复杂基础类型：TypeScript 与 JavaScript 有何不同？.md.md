# 03 | 复杂基础类型：TypeScript 与 JavaScript 有何不同？

## 数组

因为 TypeScript 的数组和元组转译为 JavaScript 后都是数组，所以这里我们把数组和元组这两个类型整合到一起介绍，也方便你更好地对比学习。

## 数组类型（Array）

使用 [] 的形式定义数组类型，如下代码所示：

```ts
/** 子元素是数字类型的数组 */
let arrayOfNumber: number[] = [1, 2, 3];

/** 子元素是字符串类型的数组 */
let arrayOfString: string[] = ['x', 'y', 'z'];
```

使用 Array 泛型（在第 10 讲会详细介绍泛型）定义数组类型，如下代码所示：

```ts
/** 子元素是数字类型的数组 */
let arrayOfNumber: Array<number> = [1, 2, 3];

/** 子元素是字符串类型的数组 */
let arrayOfString: Array<string> = ['x', 'y', 'z'];
```



以上两种定义数组类型的方式虽然本质上没有任何区别，但是我更推荐使用 [] 这种形式来定义。一方面可以避免与 JSX 的语法冲突，另一方面可以减少不少代码量。

## 元组类型（Tuple）

在 TypeScript 中，元组（Tuple）是一种特殊的数组类型，元组最重要的特性是可以限制数组元素的个数和类型，它特别适合用来实现多值返回。它允许指定数组中每个元素的类型和数量。与普通数组不同的是，元组中的每个元素可以具有不同的类型。

元组的语法类似于数组，定义元组时需要为每个元素固定类型。例如，下面是一个包含两个元素的元组，第一个元素是字符串类型，第二个元素是数字类型：

```ts
let myTuple: [string, number] = ['hello', 123];
```

在上面的代码中，我们使用了圆括号来定义一个元组类型 `[string, number]`，并将其赋值给变量 `myTuple`。该元组包含两个元素，第一个元素是字符串类型 `'hello'`，第二个元素是数字类型 `123`。

可以使用索引访问元组中的元素，就像访问普通数组一样。例如，要访问元组中的第一个元素，可以使用索引 `0`，如下所示：

```ts
let myTuple: [string, number] = ['hello', 123];
let myString: string = myTuple[0]; // 'hello'
```

元组在某些情况下非常有用，例如当需要在函数中返回多个值时。使用元组，可以将多个值打包成一个值，并将其作为函数的返回值返回。例如：

```ts
function getPerson(): [string, number] {
  return ['Alice', 30];
}

let [name, age] = getPerson();
console.log(name); // 'Alice'
console.log(age); // 30
```

上面的代码中，我们定义了一个名为 `getPerson` 的函数，该函数返回一个包含两个元素的元组，第一个元素是字符串类型，第二个元素是数字类型。我们还使用了解构赋值语法，将元组中的两个元素分别赋值给变量 `name` 和 `age`。

### 元组和数组的区别

```ts
let myArray: string[] = ['hello', 'world'];
let myTuple: [string, number] = ['hello', 123];
```

在 TypeScript 3.1 及更高版本中，元组支持使用 `push` 方法向元组中添加元素。例如，可以像下面这样向元组 `myTuple` 中添加一个新元素：

```ts
let myTuple: [string, number] = ['hello', 123];
myTuple.push(456);
console.log(myTuple); // ['hello', 123, 456]
```

![image-20230512160557404](https://s2.loli.net/2023/05/12/ylXgoTFVhr1zumw.png)
