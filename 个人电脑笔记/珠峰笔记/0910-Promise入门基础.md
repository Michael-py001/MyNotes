[TOC]



## Promise

> Promise是ES6新增的内置类：承诺模式，主要用来规划异步编程的；解决回调地狱。
>
> https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise

### MDN实例代码

#### 创建Promise

`Promise` 对象是由关键字 `new` 及其构造函数来创建的。该构造函数会把一个叫做“处理器函数”（executor function）的函数作为它的参数。这个“处理器函数”接受两个函数——`resolve` 和 `reject` ——作为其参数。当异步任务顺利完成且返回结果值时，会调用 `resolve` 函数；而当异步任务失败且返回失败原因（通常是一个错误对象）时，会调用`reject` 函数。

```js
const myFirstPromise = new Promise((resolve, reject) => {
  // ?做一些异步操作，最终会调用下面两者之一:
  //
  //   resolve(someValue); // fulfilled
  // ?或
  //   reject("failure reason"); // rejected
});
```

想要某个函数?拥有promise功能，只需让其返回一个promise即可。

```js
function myAsyncFunction(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open("GET", url);
    xhr.onload = () => resolve(xhr.responseText);
    xhr.onerror = () => reject(xhr.statusText);
    xhr.send();
  });
};
```

#### 基础示例

```js
let myFirstPromise = new Promise(function(resolve, reject){
    //当异步代码执行成功时，我们才会调用resolve(...), 当异步代码失败时就会调用reject(...)
    //在本例中，我们使用setTimeout(...)来模拟异步代码，实际编码时可能是XHR请求或是HTML5的一些API方法.
    setTimeout(function(){
        resolve("成功!"); //代码正常执行！
    }, 250);
});

myFirstPromise.then(function(successMessage){
    //successMessage的值是上面调用resolve(...)方法传入的值.
    //successMessage参数不一定非要是字符串类型，这里只是举个例子
    console.log("Yay! " + successMessage);
});
```



### 回调地狱

> 回调函数嵌套回调函数

#### 回调函数

> 把一个函数作为值，传递给另外一个函数，在另外一个函数执行的时候，再把传递进来的函数进行处理

##### 基于Promise解决回调地狱

```js
function queryInfo() {
    return new Promise(resolve => {
        $.ajax({
            url: '/api/info',
            success: function (result) {
                resolve(result);
            }
        });
    });
}

function queryScore(id) {
    return new Promise(resolve => {
        $.ajax({
            url: `/api/score?id=${id}`,
            success: function (result) {
                resolve(result);
            }
        });
    });
}

function queryPaiMing(val) {
    return new Promise(resolve => {
        $.ajax({
            url: `/api/paiming?val=${val}`,
            success: function (result) {
                resolve(result);
            }
        });
    });
}

// 基于Promise解决回调地狱
queryInfo().then(result => {
    return queryScore(result.id);
}).then(result => {
    return queryPaiMing(result.chinese);
}).then(result => {
    // 获取的排名信息
});
```

##### 基于async/await更优雅的方式解决回调地狱

```js
(async function () {
    let result = await queryInfo();
    result = await queryScore(result.id);
    result = await queryPaiMing(result.chinese);
    // 获取的排名信息
})();
```

## Promise详解

**语法：**

`let p1 = new Promise([executor])`

`new Promise( function(resolve, reject) {...} /* executor */  );`

- Promise是一个内置类

- p1是类的一个实例

- `executor`是回调函数(必须要传的)

  > executor
  >
  > executor是带有 `resolve` 和 `reject` 两个参数的函数 。Promise构造函数执行时立即调用`executor` 函数， `resolve` 和 `reject` 两个函数作为参数传递给`executor`（executor 函数在Promise构造函数返回所建promise实例对象前被调用）。`resolve` 和 `reject` 函数被调用时，分别将promise的状态改为*fulfilled（*完成）或rejected（失败）。executor 内部通常会执行一些异步操作，一旦异步操作执行完毕(可能成功/失败)，要么调用resolve函数来将promise状态改成*fulfilled*，要么调用`reject` 函数将promise的状态改为rejected。如果在executor函数中抛出一个错误，那么该promise 状态为rejected。executor函数的返回值被忽略。

#### Promise本身是同步的(用来管理异步编程)

```js
let p1 = new Promise((resolve, reject) => {
    console.log('OK'); //1 先输出
});
console.log('NO'); //2  后输出
```

**创建一个Promise实例：**`let p1 = new Promise((resolve,reject)=>{resolve("ok")})`

**实例上的状态/值:**

- `[[PromiseState]]` Promise状态
  
  - pending 初始状态
  - fulfilled/resolved 成功状态(一般指的是异步请求成功)
  - rejected 失败状态(一般指的是异步请求失败)
- `[[PromiseResult]]` Promise结果/值
  - 初始值是undefined
  - 不论成功还是失败，成功的结果或者失败的原因都会赋值给他
  
  ![image-20200914154448440](https://i.loli.net/2020/09/14/8eYHnV4pP7UWhST.png)

**new Promise时会立即把executor函数执行:**

> executor是带有 `resolve` 和 `reject` 两个参数的函数，`resolve` 和 `reject` 两个函数作为参数传递给`executor` 

- `resolve`函数：函数执行会修改Promise实例状态为成功，传递的值就是`[[PromiseResult]]`，回调函数的`resolve/reject`必须要传了函数内才能使用
  ![image-20200914192734160](https://i.loli.net/2020/09/14/wnOAHkxPqb8JrF1.png)

- `reject`函数：函数执行会修改Promise实例状态为失败，传递的值就是`[[PromiseResult]]`，返回失败状态的实例，但是没有做失败的处理，浏览器控制台有报错（但是不会影响其他代码执行。）

  ![image-20200914193815858](https://i.loli.net/2020/09/14/AgdTtbJZova3LjC.png)

- 如果executor里的回调函数没有执行`resolve`或者`reject`函数，则`Promise`实例的状态值还是`pending`。

  ![image-20200914192357629](https://i.loli.net/2020/09/14/2NZBOXbdxUPrnSz.png)

- 我们一般会在executor函数中管理一个异步操作：

  - **异步操作成功-->执行resolve**，让实例状态变为成功，并且获得成功的结果
  - **异步操作失败，执行reject**，让实例状态变为失败，并获取失败的原因(reason)

- 只要Promise的状态从`pending`变为`fulfilled`或者`rejected`，则状态不能再变化了。

  ```js
  let p1 = new Promise((resolve, reject) => {
      resolve(100); //改变状态后，下面再改变状态就无用了
      reject(0);
  });
  ```

### Promise.prototype上的方法

#### `then([A],[B])` 

> 基于then方法存放的两个回调函数A / B，当Promise状态成功则执行 A，失败则执行B，并且把[[PromiseResult]]的值传递给对应的函数-->[返回值]

#### `catch`

> 捕获错误，相当于then第一个参数不传，只接收失败信息

#### `finally`  

> 无论结果失败还是成功，都会执行

<img src="https://i.loli.net/2020/09/12/Q7GkneNuPz5oYTX.png" alt="image-20200912230732840" style="zoom:80%;" />

因为 `Promise.prototype.then` 和 `Promise.prototype.catch` 方法返回promise 对象， 所以它们可以被链式调用。

![img](https://mdn.mozillademos.org/files/8633/promises.png)

### 执行then方法的过程

​	执行then方法是为了把当前实例成功执行的A以及失败执行的B，会返回一个新的Promise实例-->P2；

```js
let p1 = new Promise((resolve, reject) => {
    // 管理一个异步操作：根据需求控制promise成功还是失败
    setTimeout(() => {
        let ran = Math.random();
        ran < 0.5 ? reject('NO') : resolve('OK');
    }, 1000);
});
let p2 = p1.then(result => {
    console.log(`成功了 -> ${result}`);
    return result + '@@';
}, reason => {
    console.log(`失败了 -> ${reason}`);
    return reason + '@@';
});
```

- **成功则执行then(A,B)中的A函数:  实例是resolved成功状态 **
  ![image-20200914200621026](https://i.loli.net/2020/09/14/8Hypdbm54VsOUl6.png)
- **失败则执行then(A,B)中的B函数，实例状态也是成功的resolved(因为没有报错，默认会成功)**
  ![image-20200914201140408](https://i.loli.net/2020/09/14/Xx7pSAloL6wnyPr.png)
- **如果要返回成功或失败状态的实例，可以自己手动加一个reject或resolve**
  ![image-20200914201432029](https://i.loli.net/2020/09/14/f8V4p5A6LrM7WCg.png)

- **P2的状态/结果 -->由上一个实例p1基于then存放的A/B函数决定**。
  - 不论是A还是B执行，**只要执行不报错，则p2的状态都是成功的**，反之执行报错就是失败的
  - P2的结果是A或者B函数执行的返回值，再或者是报错原因
- **特殊情况**：如果A/B返回的是一个新的Promise实例，则返回的Promise实例的成功失败以及结果，直接决定了P2的状态和结果。

​        p1的成功和失败也会受到executor执行是否报错影响，执行报错了，则p1的状态就是失败的，promiseResult的值是失败的原因；如果执行不报错，再看执行的是resolve/reject。

### then()中的"顺延"

`then([A],[B])`：如果其中一个函数没有传递，则会“顺延”到下一个then中

- **成功顺延resolve**：本来resolve是成功的，应该传递给then中的A,但是`[A]`没有传或者是null，则找下一个then中的A函数

  ```js
  Promise.resolve(10).then(null, reason => {
      console.log(`失败了 -> ${reason}`);
      return reason / 10;
  }).then(result => {
      console.log(`成功了 -> ${result}`); //成功->10
      return result * 10;
  }, reason => {
      console.log(`失败了 -> ${reason}`);
      return reason / 10;
  });
  ```

  **原理**：第一个A没有传，默认会放这样一个函数进去-->`result =>{return result}`，传进去什么就返回什么，代码不报错，进入下一个then中。

  

- **失败顺延reject**：本来reject是传给then中的B，但是`[B]`没有传递或者是null，则顺延到下一个then中的B

  ```js
  Promise.reject(10).then(null, null).then(result => {
      console.log(`成功了 -> ${result}`);
      return result * 10;
  }, reason => {
      console.log(`失败了 -> ${reason}`);//-->失败
      return reason / 10;
  });
  ```

  **原理**: B没传，默认传一个函数->`reason=>{return Promise.reject(reason);}`，虽然没有报错，但是当前的then的状态由返回的Promise实例的状态决定，这里是reject返回，则一定是失败，则下一个then的状态也是失败，进入B中。

### 真实项目中的then

真实项目中：then只是处理成功的(因为第二个函数B没传，只会把成功的传到then中的A)，  catch处理失败，一般写在最后(因为catch相当于then(null,B)A接收成功的函数没传，只能传到B中)，所以无论有多少个then，都会把错误`reason`传到`catch`中。

```js
Promise.resolve(10).then(result => {
    console.log(`成功了 -> ${result}`);
    return result * a;
}).then(result => {
    console.log(`成功了 -> ${result}`);
    return result * a;
}).catch(reason => {
    console.log(`失败了 -> ${reason}`);
}); 
```

### catch为了解决报错，捕获错误

**返回失败状态的实例，但是没有做失败的处理，浏览器控制台有报错（但是不会影响其他代码执行**。

```js
Promise.reject(10).then(result => {
    console.log(`成功了 -> ${result}`);
    return result * 10;
})
```

![image-20200913010402519](https://i.loli.net/2020/09/13/Ey6D7PjIrYiGvhA.png)

![image-20200914193815858](https://i.loli.net/2020/09/14/Pg1T76z5RZaNYdM.png)

**加上catch解决这个问题**

#### `catch(reason)`相当于`then(null,reason=>{})`

catch相当于then(null,B)第一个参数不传，把失败的顺延到B中

```js
Promise.reject(10).then(result => {
    console.log(`成功了 -> ${result}`);
    return result * 10;
}).catch(reason => {});
setTimeout(() => {
    console.log('OK');
}, 1000);
```



### Promise对象上的方法

- `resolve(value)` 返回一个状态为成功的Promise实例
- `reject(reason)` 返回一个状态为失败的Promise实例
- `all(iterable)`  
- `Promise.allSettled(iterable)`
- `race.(iterable)` 

###  `Promise.all`

> Promise.all管理多个promise实例，返回一个总的promise实例 P 
>
> 这个方法返回一个新的promise对象，该promise对象在iterable参数对象里所有的promise对象都成功的时候才会触发成功，一旦有任何一个iterable里面的promise对象失败则立即触发该promise对象的失败。这个新的promise对象在触发成功状态以后，会把一个包含iterable里所有promise返回值的数组作为成功回调的返回值，顺序跟iterable的顺序保持一致；如果这个新的promise对象触发了失败状态，它会把iterable里第一个触发失败的promise对象的错误信息作为它的失败错误信息。Promise.all方法常被用于处理多个promise对象的状态集合。（可以参考jQuery.when方法---译者注）

- 当所有实例都为成功，P状态才是成功（结果是一个数组，按照顺序依次存储了每一个实例成功的结果）
- 只要有一个实例失败的，P状态就是失败（结果就是当前实例失败的原因

```js
Promise.all([promise1,promise2,...]).then(result=>{}).catch(reason=>{})
```

### [`Promise.allSettled(iterable)`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled)

等到所有promises都完成（每个promise返回成功或失败）。
返回一个promise，该promise在所有promise完成后完成。并带有一个对象数组，每个对象对应每个promise的结果。

### `Promise.race`

> Promise.race也是管理多个promise实例，但是它属于看所有实例谁先有结果（不论是失败还是成功），总实例以它为主

### [`Promise.reject(reason)`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/reject)

返回一个状态为失败的Promise对象，并将给定的失败信息传递给对应的处理方法

### [`Promise.resolve(value)`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve)

返回一个状态由给定value决定的Promise对象。如果该值是thenable(即，带有then方法的对象)，返回的Promise对象的最终状态由then方法执行决定；否则的话(该value为空，基本类型或者不带then方法的对象),返回的Promise对象状态为fulfilled，并且将该value传递给对应的then方法。通常而言，如果你不知道一个值是否是Promise对象，使用Promise.resolve(value) 来返回一个Promise对象,这样就能将该value以Promise对象形式使用。

### [`Promise.any(iterable)`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/any)

接收一个Promise对象的集合，当其中的一个promise 成功，就返回那个成功的promise的值。

### 例题

成功时的输出：

```
new Promise((resolve, reject) => {
    resolve(10);
    // reject(20);
}).then(result => {
    console.log(`成功了 -> ${result}`); //输出10
    return result * 10; //返回100
}, reason => {
    console.log(`失败了 -> ${reason}`);
    return reason / 10;
}).then(result => {
    console.log(`成功了 -> ${result}`); //输出100
    return Promise.reject(result * 10); //返回错误1000
}, reason => {
    console.log(`失败了 -> ${reason}`);
    return reason / 10;
}).then(result => {
    console.log(`成功了 -> ${result}`);
    return result * 10;
}, reason => {
    console.log(`失败了 -> ${reason}`); //输出1000
    return reason / 10; //返回100
});
```

失败时的输出：

```js
new Promise((resolve, reject) => {
    reject(20);
}).then(result => {
    console.log(`成功了 -> ${result}`);
    return result * 10;
}, reason => {
    console.log(`失败了 -> ${reason}`); //输出20
    return reason / 10; //返回2
}).then(result => {
    console.log(`成功了 -> ${result}`); //没有报错，会传到A，输出2
    return Promise.reject(result * 10); //返回错误20
}, reason => {
    console.log(`失败了 -> ${reason}`);
    return reason / 10;
}).then(result => {
    console.log(`成功了 -> ${result}`);
    return result * 10;
}, reason => {
    console.log(`失败了 -> ${reason}`); //输出20
    return reason / 10; //返回2
});
```

