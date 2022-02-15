## Promise完整版手写源码

```js
(function () {
    // 构造函数
    function MyPromise(executor) {
        if (typeof executor !== "function") throw new TypeError('MyPromise resolver undefined is not a function');

        // 每一个MyPromise实例应该具备这几个属性
        this.MyPromiseState = "pending";
        this.MyPromiseValue = undefined;
        this.resolveFunc = function () {};
        this.rejectFunc = function () {};
        // 保存当前实例的this，不然change执行，里面的this是window
        var _this = this;
        // 修改它的状态/值，并且通知指定的方法执行
        var change = function change(state, value) {
            if (_this.MyPromiseState !== "pending") return;
            _this.MyPromiseState = state;
            _this.MyPromiseValue = value;
            setTimeout(function () {
                state === "fulfilled" ?
                    _this.resolveFunc(_this.MyPromiseValue) :
                    _this.rejectFunc(_this.MyPromiseValue);
            }, 0);
        };
        var resolve = function resolve(result) {
            change('fulfilled', result);
        };
        var reject = function reject(reason) {
            change('rejected', reason);
        };

        try {
            executor(resolve, reject);
        } catch (err) {
            change('rejected', err);
        }
    }

    // 构造函数原型
    MyPromise.prototype = {
        constructor: MyPromise,
        then: function (resolveFunc, rejectFunc) {
            //_this代表的是当前实例，下面return MyPromise里的参数resolve/reject函数里的this不是当前的实例
            var _this = this;

            // 参数不传递：顺延
            if (typeof resolveFunc !== "function") {
                resolveFunc = function resolveFunc(result) {
                    return MyPromise.resolve(result);
                };
            }
            if (typeof rejectFunc !== "function") {
                rejectFunc = function rejectFunc(reason) {
                    return MyPromise.reject(reason);
                };
            }

            // 每一次执行THEN方法都会返回一个新的MyPromise实例
            //   + resolve/reject 是控制新返回实例的成功和失败的
            //   + 把传递进来的需要执行的A:resolveFunc/B:rejectFunc进行包装处理
            return new MyPromise(function (resolve, reject) {
                //_this是实例中的this，给_this.resolveFunc赋值一个包装函数，这个函数里会执行成功/失败需要执行的函数-->then()中传进来的resolveFun/rejectFunc
                _this.resolveFunc = function (result) {
                    try {
                        // 从then(resolveFunc,rejectFunc)中传进来的resolveFunc，是成功后需要执行的函数，这里要接收函数执行的结果，成功则resolve(x),失败则reject(x),x参数就是前面传给Promise的value值-->_this.MyPromiseValue
                        // 执行不报错，则新返回的实例是成功的（特殊：方法返回的是一个新的MyPromise实例，则新实例的状态和结果决定返回实例的状态和结果）
                        var x = resolveFunc(result);
                        x instanceof MyPromise ?
                            //如果返回的是一个Promise函数如：【return MyPromise.reject('P1 OK');】,则执行then(resolve, reject),如果返回的是成功的，则执行resolve,失败则执行reject，而resolve和reject是哪里来的呢？就是上面new MyPromise(executor)，executor里的resolve/reject。返回的x是成功的,x.then(resolve,reject)就会执行resolve函数，执行了之后，就会通知上面的MyPromise状态变为成功。反之，返回的x是失败的，则执行x.then(resolve,reject)中的reject函数，通知MyPromise状态变为失败。
                            x.then(resolve, reject) :
                            resolve(x);
                    } catch (err) {
                        // 执行报错，则新返回的实例是失败的
                        reject(err);
                    }
                };
                _this.rejectFunc = function (reason) {
                    try {
                        var x = rejectFunc(reason);
                        x instanceof MyPromise ?
                            x.then(resolve, reject) :
                            resolve(x);
                    } catch (err) {
                        reject(err);
                    }
                };
            });
        },
        catch: function (rejectFunc) {
            // xxx.catch([function]) => xxx.then(null,[function])
            return this.then(null, rejectFunc);
        }
    };

    // 普通对象
    MyPromise.resolve = function resolve(result) {
        return new MyPromise(function (resolve) {
            resolve(result);
        });
    };
    MyPromise.reject = function reject(reason) {
        return new MyPromise(function (_, reject) {
            reject(reason);
        });
    };
    MyPromise.all = function all(promiseArr) {
        // var _this = this;
        return new MyPromise(function (resolve, reject) {
            var index = 0, //记数
                results = []; //结果集合

            // 都成功后控制返回的实例也是成功
            var fire = function () {
                if (index >= promiseArr.length) {
                    resolve(results);
                }
            };

            // 循环依次处理每一个实例
            for (var i = 0; i < promiseArr.length; i++) {
                // 基于闭包机制存放每一轮循环的索引，保证每一轮循环的i都是正确的
                (function (i) {
                    var item = promiseArr[i];
                    if (!(item instanceof MyPromise)) {
                        results[i] = item;
                        index++;
                        fire();
                        return;
                    }
                    item.then(function (result) {
                        results[i] = result;
                        index++;
                        fire();
                    }).catch(function (reason) {
                        // 只要有一个是失败的，整体都是失败的
                        reject(reason);
                    });
                })(i);
            }
        });
    };

    // 让其支持浏览器导入和CommonJS/ES6Module模块导入规范
    if (typeof window !== "undefined") {
        window.MyPromise = MyPromise;
    }
    if (typeof module === "object" && typeof module.exports === "object") {
        module.exports = MyPromise;
    }
})();
```

