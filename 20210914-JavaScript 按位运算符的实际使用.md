# JavaScript 按位运算符~的实际使用

### 按位非 ~运算符

> 它将整数转换为 -(N+1) 值

先看输出效果

```js
if (~res.provider.indexOf('weixin')) {...} //~符号
```

![image-20210914185505642](https://i.loli.net/2021/09/14/NodViSJe8z9RnBC.png)

`indexOf`如果找不到则返回-1

单个按位非运算符输出结果

```js
~2 === -3; //true
~1 === -2; //true
~0 === -1; //true
~-1 === 0; //true
```

利用此运算符**功能**的最实用方法是将**其用作`Math.floor()`函数的替代品**

双按位非运算符的使用示例

```js
~~2 === Math.floor(2); //true, 2
~~2.4 === Math.floor(2); //true, 2
~~3.9 === Math.floor(3); //true, 3
```

速度比Math.floor快，就是可读性差
