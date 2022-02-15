### compose函数

面试题

```js
在函数式编程当中有一个很重要的概念就是函数组合， 实际上就是把处理数据的函数像管道一样连接起来， 然后让数据穿过管道得到最终的结果。 例如：
    const add1 = (x) => x + 1;
    const mul3 = (x) => x * 3;
    const div2 = (x) => x / 2;
    div2(mul3(add1(add1(0)))); //=>3

    而这样的写法可读性明显太差了，我们可以构建一个compose函数，它接受任意多个函数作为参数（这些函数都只接受一个参数），然后compose返回的也是一个函数，达到以下的效果：
    const operate = compose(div2, mul3, add1, add1)
    operate(0) //=>相当于div2(mul3(add1(add1(0)))) 
    operate(2) //=>相当于div2(mul3(add1(add1(2))))
​
    简而言之：compose可以把类似于f(g(h(x)))这种写法简化成compose(f, g, h)(x)，请你完成 compose函数的编写    
```

三个函数：

```js
const add1 = x => x + 1;
const mul3 = x => x * 3;
const div2 = x => x / 2;
```

#### 方案一

```js
function compose(...funcs) {
    // funcs存储的是最后需要处理的函数集合
    return function operate(x) {
        // x存储初始第一个函数执行需要的实参
        if (funcs.length === 0) return x;
        if (funcs.length === 1) return funcs[0](x);

        // 把数据倒过来或者用reduceRight
        funcs.reverse();
        let first = funcs[0](x);
        funcs = funcs.slice(1);
        return funcs.reduce((result, item) => {
            return item(result);
        }, first);
    };
}
```

#### 方案二

```js
function compose(...funcs) {
    return function operate(x) {
        if (funcs.length === 0) return x;
        if (funcs.length === 1) return funcs[0](x);

        let n = 0;
        funcs.reverse();
        return funcs.reduce((result, item) => {
            if ((++n) === 1) {
                // 第一次处理 result第一个函数 item第二个函数
                return item(result(x));
            }
            return item(result);
        });
    };
}
```

#### redux源码中的处理方案

```js
function compose(...funcs) {
    if (funcs.length === 0) {
        return x => {
            return x;
        };
    }

    if (funcs.length === 1) {
        return funcs[0];
    }

    return funcs.reduce((a, b) => {
        return (...args) => {
            return a(b(...args));
        };
    });
}

let operate = compose(div2, mul3, add1, add1);
console.log(operate(0)); //=>3

operate = compose();
console.log(operate(0)); //=>0

operate = compose(add1);
console.log(operate(0)); //=>1
```

