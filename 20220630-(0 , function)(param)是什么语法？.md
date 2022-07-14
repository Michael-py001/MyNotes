# 20220630-(0 , function)(param)是什么语法？

> 参考文章：
>
> [javascript_冷知识之_(0, function)(param)_一把健的博客-CSDN博客_(0, function)](https://blog.csdn.net/qq_39446719/article/details/103838706)

在看uniapp的编译源码时，发现很多地方使用了(0,func)的语法：

![image-20220630150914300](https://s2.loli.net/2022/06/30/Xxc1FKuTAgHzRUB.png)

其本质是 `js` 的 [逗号操作符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comma_Operator)
**逗号操作符** 对它的每个操作对象求值（从左至右），然后返回最后一个操作对象的值。

> [逗号操作符 - JavaScript | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comma_Operator)

例：

```js
let x = 1;

x = (x++, x);

console.log(x);
// expected output: 2

x = (2, 3);

console.log(x);
// expected output: 3

```

那么回到这段代码中，进行调用时则可以等价于：

```js
const dog = {
    run() {
        console.log(this);
        console.log('I\'m running');
    }
};
dog.run();
(0, dog.run)();

```

```js
(0, dog.run)();

// 左边先用逗号操作符从左到右执行并取值，最终返回
dog.run: function run() {
	console.log(this);
	console.log('I\'m running');
}
// 等价于如下，此时的 this 就是 全局对象 了
(function run() {
	console.log(this);
	console.log('I\'m running');
})();

```

