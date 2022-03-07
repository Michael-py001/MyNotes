## Promise简版手写源码

```js
function MyPromise(executor) {
    // executor必须是一个可执行的函数
    if (typeof executor !== "function") throw new TypeError('MyPromise resolver undefined is not a function');

    // 每一个MyPromise实例应该具备这几个属性
    this.MyPromiseState = "pending";
    this.MyPromiseValue = undefined;
    // 成功后要执行的方法
    this.resolveFunc = function () {};
    //失败后要执行的方法
    this.rejectFunc = function () {};

    // 保存当前实例的this，不然change执行，里面的this是window
    var _this = this;

    // 修改它的状态/值，并且通知指定的方法执行
    var change = function change(state, value) {
        // 改变状态信息是同步执行的（executor立即执行就会改变状态）
        // 一但状态改变后，则不能再更改为其他状态了
        if (_this.MyPromiseState !== "pending") return;
        _this.MyPromiseState = state;
        _this.MyPromiseValue = value;

        //这里是需要then传进方法赋值后才执行
        // 更改状态和value是同步的，但是通知方法执行是异步的（因为必须要等到then执行完，方法放置好之后再通知方法执行）
        //因为new一个myPromise时，会立即把executor函数执行，this.resolveFunc还没有被赋值，还是初始状态，因此要等then(resovle,reject)把方法传进来赋值后才能执行成功/失败需要执行的方法-->设置成宏任务(微任务自己搞不了)，可以设置一个定时器，宏任务会在同步任务执行完后才会回来执行
        setTimeout(function () {
            // 如果传进来的state是fulfilled(成功)，则执行this.resolveFunc方法
            //否则就执行失败后执行的方法：this.rejectFunc
            //执行的函数，其实就是then中传进来的函数
            state === "fulfilled" ?
                _this.resolveFunc(_this.MyPromiseValue) :
                _this.rejectFunc(_this.MyPromiseValue);
        }, 0);
    };
    // 执行resolve，会修改状态/值和通知方法执行
    var resolve = function resolve(result) {
        change('fulfilled', result);
    };
    // 执行reject，会修改状态/值和通知方法执行
    var reject = function reject(reason) {
        change('rejected', reason);
    };

    // executor执行报错,也会让实例的状态变为失败，例如两个参数都不传会报错
    try {
        executor(resolve, reject);
    } catch (err) {
        // 报错了通知改变状态信息，失败的原因是报错信息
        change('rejected', err);
    }
}
// 重写MyPromise的原型，添加then方法
MyPromise.prototype = {
    constructor: MyPromise,
    // 执行.then，会把传进去的resolveFunc/rejectFunc 赋值给myPromise中的this.resolveFunc/this.rejectFunc
    //resolveFunc/rejectFunc就是成功/失败后要做的事情的函数
    then: function then(resolveFunc, rejectFunc) {
        // 参数不传递：顺延到下一个then（需要实现增量机制后才能实现，完整版）
        if (typeof resolveFunc !== "function") {
            //resolveFunc(result)里的result,执行时传进去的就是_this.MyPromiseValue 的value值
            resolveFunc = function resolveFunc(result) {
                // return result //也是可以的，下面的语义化更强点
                return MyPromise.resolve(result);
            };
        }
        if (typeof rejectFunc !== "function") {
            rejectFunc = function rejectFunc(reason) {
                return MyPromise.reject(reason);
            };
        }
        // this就是实例中的this,赋值函数给this.resolveFunc
        this.resolveFunc = resolveFunc;
        this.rejectFunc = rejectFunc;
    }
};
// MyPromise当做对象上的方法
// 返回一个状态为成功的MyPromise实例
MyPromise.resolve = function resolve(result) {
    // 返回一个成功的状态，只需要new一个MyPromise(executor)，里面的executor传一个成功函数，也就是把resolve传进去
    //value值就是传进去的result
    return new MyPromise(function (resolve) {
        resolve(result);
    });
};
// 返回一个状态为失败的MyPromise实例
MyPromise.reject = function reject(reason) {
    //把new MyPromise的参数executor传一个失败的reject进去，状态就变成失败了
    //value值就是传进去的reason
    return new MyPromise(function (_, reject) {
        reject(reason);
    });
};

var p1 = new MyPromise(function (resolve, reject) {
    // resolve('OK');
    reject('NO');
});
// 需要实现增量机制后才能实现
p1.then(null
    /*function (result) {
         MyPromise.resolve(result);
    }*/
    ,
    null
    /*function (reason) {
        return MyPromise.reject(reason);
    }*/
);
```

