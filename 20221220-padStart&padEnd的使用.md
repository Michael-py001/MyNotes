# padStart&padEnd的使用

> [String.prototype.padStart() - JavaScript | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/padStart)

## String.prototype.padStart()

**`padStart()`** 方法用另一个字符串填充当前字符串（如果需要的话，会重复多次），以便产生的字符串达到给定的长度。从当前字符串的左侧开始填充。

```js
const str1 = '5';

console.log(str1.padStart(2, '0'));
// expected output: "05"

const fullNumber = '2034399002125581';
const last4Digits = fullNumber.slice(-4);
const maskedNumber = last4Digits.padStart(fullNumber.length, '*');

console.log(maskedNumber);
// expected output: "************5581"
```

也可以用来填充时间，比如格式需要固定两位的，不足10的用0填充的场景：

```js
//getRestTime返回的格式是：day + '天' + hour + '时' + minute + '分' + second + '秒'
// 或者：[day, hour, minute, second]
getRestTime(time, false).map(v => v.toString().padStart(2, '0'))
```

## String.prototype.padEnd()

**`padEnd()`** 方法会用一个字符串填充当前字符串（如果需要的话则重复填充），返回填充后达到指定长度的字符串。从当前字符串的末尾（右侧）开始填充。

```js
const str1 = 'Breaded Mushrooms';

console.log(str1.padEnd(25, '.'));
// expected output: "Breaded Mushrooms........"

const str2 = '200';

console.log(str2.padEnd(5));
// expected output: "200  "
```

