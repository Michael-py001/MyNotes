# 20211028-some, every的用法

> 参考文章： https://www.cnblogs.com/redsquirrel/p/11546534.html
>

## some

some会一次遍历数组的每一个元素，遇到满足条件的之后，后面不再遍历，返回true。

```js
let arr = [3,8,9,16,25]
const check = (age) => {
    console.log(age)
    return age > 5
}

let isGood = arr.some(check)
console.log(isGood) //true
```

## every

every会遍历每一个元素，遇到有一个元素不满足，立即返回false，后面的元素不再返回

```js
let arr = [3,8,9,16,25]
const check = (age) => {
    console.log(age)
    return age > 5
}

let isGood = arr.every(check)
console.log(isGood) //false
```

