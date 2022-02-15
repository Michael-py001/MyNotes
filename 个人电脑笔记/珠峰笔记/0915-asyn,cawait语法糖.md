## async/await：ES7中新提供的语法糖

> async修饰一个函数：保证函数返回的是一个promise实例。

- 和then很相似，函数执行不报错，返回成功的promise实例，报错返回的是失败的promise实例
- return的值或者报错的原因就是promise实例的结果
- 如果return的是一个新的promise实例，则新实例的结果影响返回promise的结果

- [x] async最常见的应用，是为了修饰函数，让函数中可以使用await（想要使用await，所在的函数必须是基于async修饰的）

- [x] await [promise实例]  等待promise实例状态为成功过的时候，再继续执行await下面的代码

```js
function func() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve("成功了")
        }, 1000)
    })
}

async function fn() {
    news = await func() //news就是promise实例成功的后的结果
    console.log(news)
}
fn()
```

### await报错处理

如果await后面的promise实例返回的是失败的，浏览器会报错，而且后面的代码也不会执行了

```js
(async function () {
    let x = await 10;
    console.log(x); //10
    let y = await Promise.resolve(20);
    console.log(y); //20
    let z = await Promise.reject(30);//浏览器报错
    console.log(z); //不执行了

})();
```

解决方案是使用`try..catch`捕获异常

```js
(async function () {
    let x = await 10;
    console.log(x); //10

    let y = await Promise.resolve(20);
    console.log(y); //20
    let z = await Promise.reject(30);
    console.log(z);
    try {
        let z = await Promise.reject(30); //await后面的promise如果是失败的，则当前函数中await下面的代码都不会执行
        console.log(z);
    } catch (err) {
        // 基于try catch异常捕获，可以捕获到await后面的promise实例是失败状态下的失败信息(浏览器控制台不会再报错了)
        console.log(err); //30
    }
})();
```

### await运行机制

在一个函数内遇到`await`，后面紧跟着的函数是同步执行，先执行这个函数，除此之外：

- 遇到`await`，下面的代码整体放入到“`微任务`”队列中;
- 当JS渲染线程把同步任务都执行完后，并且也知道await后面的promise是成功状态，才会把之前存放的微任务拿出来执行

**例题一：**

```js
 (async function () {
    await 10; //遇到await下面代码先放置在微任务队列中
    console.log('10'); //2
})();
console.log('OK'); //1  
```

如果遇到两个`await`：

> 如果遇到两个`await`，会整体把第一个`await`下面的代码列为微任务1，返回成功后，执行下面的微任务，遇到`await`，再次把await下面的代码列为微任务2，返回成功后执行下面的代码

```js
(async function () {
    await 10;
    console.log(10);//2

    await 20;
    console.log(20);//3
})();
console.log('OK'); //1 
```

**例题二：async后的函数返回的不是promise时**

> async 后面的函数执行完，async接收的是一个成功的promise实例，如果后面的函数里返回的不是promise，那么async接收的也是一个promise，只不过值为undefind

```js
async function async1() {
    console.log('async1 start'); //2
    await async2(); //async2--3
    console.log('async1 end'); //6 微任务1
}
async function async2() {
    console.log('async2');
}

console.log('script start'); //1

setTimeout(function () {
    console.log('setTimeout'); //7 宏任务1
}, 0)

async1();

new Promise(function (reject) {
    console.log('promise1'); //4
    reject("1000"); //Promise是微任务2
}).then(function () {
    console.log('promise2'); //reject微任务通知后then执行
});
console.log('script end'); //5 
```

![image-20200915112215081](https://i.loli.net/2020/09/15/w7agQkvGyElBD2H.png)

**async后的函数返回是一个promise时**

> ​		这里注意，如果await后面的函数里返回的是一个promise实例，则会首先把这个promise放入到微任务队列，等待这个promise返回成功后，await下面的代码才会执行，在这期间，同步任务会继续往下执行，如果下面又遇到微任务，就添加到微任务队列，执行完同步任务后，返回去执行第一个微任务，await接收到成功的promise后，开始执行它下面的代码。

```js
// async后是一个promise时
async function async1() {
    console.log('async1 start'); //2
    await async2(); //同步执行async2，等待返回成功后才执行下面的代码，所以下面的代码是最后放入到微任务队列里的
    console.log('async1 end'); //微任务3
}
async function async2() {
    //返回的是promise，放入微任务队列-->微任务1
    return Promise.reject("no ok").catch(reason => {
        console.log("失败原因：", reason)
    })
}

console.log('script start'); //1

setTimeout(function () {
    console.log('setTimeout'); //宏任务1
}, 0)

async1();

new Promise(function (reject) {
    console.log('promise1'); //3
    reject("1000"); //reject是微任务2
}).then(function () {
    console.log('promise2'); //微任务2
});
console.log('script end'); //4  
```

![image-20200915112138322](https://i.loli.net/2020/09/15/qYl1sSigG63LIx4.png)

**例题三：**

> 注意同步执行中的循环和宏任务等待时间

```js
function func1() {
    console.log('func1 start'); //3  第二次是--8
    return new Promise(resolve => {
        resolve('OK'); //微任务1
    });
}

function func2() {
    console.log('func2 start'); //4
    return new Promise(resolve => { //微任务2
        setTimeout(() => {
            resolve('OK'); //--下面的settimeout-9输出后才返回
        }, 10); //宏任务2
    });
}

console.log(1); //1
setTimeout(async () => {
    console.log(2); //宏任务1 --7
    await func1();
    console.log(3); //--9
}, 20);
//这里注意：循环结束后，已经过了80MS，上面的settimeout等待时间已经结束，马上执行这个宏任务
for (let i = 0; i < 90000000; i++) {} //循环大约要进行80MS左右
console.log(4); //2
func1().then(result => {
    console.log(5); //微任务1返回 --6
});
func2().then(result => {
    console.log(6); //微任务2返回（func2里是一个宏任务，放到微任务执行完后才执行）---11
});
//这个宏任务3和func2里的宏任务2，看哪个时间短，先执行哪个
setTimeout(() => {
    console.log(7); //宏任务3--10
}, 0);
console.log(8); //5
```

