# padStart和padEnd

## padStart

语法

`str.padStart(targetLength [, padString])`

### [参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/padStart#参数)

- `targetLength`

  当前字符串需要填充到的目标长度。如果这个数值小于当前字符串的长度，则返回当前字符串本身。

- `padString` 可选

  填充字符串。如果字符串太长，使填充后的字符串长度超过了目标长度，则只保留最左侧的部分，其他部分会被截断。此参数的默认值为 " "（U+0020）。

```js
const str1 = '5';

console.log(str1.padStart(2, '0'));
// expected output: "05"

'abc'.padEnd(10);          // "abc       "
'abc'.padEnd(10, "foo");   // "abcfoofoof"
'abc'.padEnd(6, "123456"); // "abc123"
'abc'.padEnd(1);           // "abc"
```

### [返回值](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/padStart#返回值)

在原字符串开头填充指定的填充字符串直到目标长度所形成的新字符串。

## padEnd

语法

`str.padEnd(targetLength [, padString])`

### [参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/padEnd#参数)

- `targetLength`

  当前字符串需要填充到的目标长度。如果这个数值小于当前字符串的长度，则返回当前字符串本身。

- `padString` 可选

  填充字符串。如果字符串太长，使填充后的字符串长度超过了目标长度，则只保留最左侧的部分，其他部分会被截断。此参数的缺省值为 " "（U+0020）。

### [返回值](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/padEnd#返回值)

在原字符串末尾填充指定的填充字符串直到目标长度所形成的新字符串。

```js
const str1 = 'Breaded Mushrooms';

console.log(str1.padEnd(25, '.'));
// expected output: "Breaded Mushrooms........"

const str2 = '200';

console.log(str2.padEnd(5));
// expected output: "200  "
```



## 实际应用

```js
// 填充月份
this.dateArray[1] = Array(12).fill(0).map((i, k) => {
    return {
        value: (k + 1).toString().padStart(2, '0'),
        name: (k + 1).toString().padStart(2, '0') + '月'
    }
})

// 输出01 02 03 ... 10 11 12
```

