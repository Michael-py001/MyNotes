### call

`fn.call([context],params1,params2,...)`

**简单说明：**把函数执行，让函数中的this指向[context]，并且把params1/params2...作为实参传递给函数

**详细说明：**

1. 首先fn基于原型链`__proto__`，找到Function.prototype.call方法，并且把call方法执行。
2. call方法中的this就是当前操作的实例fn，传递给call方法的第一个实参是未来改变fn中的this指向，剩余的实参，都是未来要依次传递给fn的参数信息
3.  call方法执行的过程中，实现了这样的处理：把fn「call中的this」执行，让fn中的this指向[context]，并且把params1/params2...作为实参传递给fn，以此达到最后的效果
4. 最后接受fn执行的返回结果，作为返回值反给外部

```js
const fn = function fn(x, y) {
    this.total = x + y;
    console.log(this);
    return this;
};
window.name = '珠峰';
let obj = {
    name: 'obj'
};

let res = fn.call(obj, 10, 20);
fn.call(10, 20); //fn: this->10 x=20 y=undefined
fn.call(); //fn: this->window  第一个参数不传或者传递的是null/undefined，在JS非严格模式下，最后fn中的this都是window（严格模式下，不传this是undefined，传递null/undefined，this也会改为对应的值）
```

### apply

`函数.apply([context],[params1,params2,...])`

**call和apply的唯一区别**：执行函数的时候，需要传递给函数的参数信息，在最开始传递给call/apply的时候，形式不一样

- **call**是需要把参数一个个传递给call，call方法内部再一个个传递给函数
- **apply**是需要把参数放在一个数组中传递给apply，但是apply内部也会帮我们把接受数组中的每一项一项项的传递给函数

### bind

**bind与call/apply的区别：**

- call/apply在执行的时候，都会立即把要操作的函数执行，并且改变它的this指向
- bind是预先处理：执行bind只是预先把函数中需要改变的this等信息存储起来（预设好），但是此时函数并不会被执行，执行bind会返回一个匿名函数，当后期执行匿名函数的时候，再去把之前需要执行的函数执行，并且改变this为预设的值

```js
const fn = function fn(x, y) {
    this.total = x + y;
    console.log(this);
    return this;
};
window.name = '珠峰';
let obj = {
    name: 'obj'
};

let anonymous = fn.bind(obj, 10, 20); //执行bind的时候,fn是不会执行
console.log(anonymous); //后期执行把返回的函数再执行，在里面才会把fn执行，并且按照预设的信息改变this等 
```

```js
需求：1s后执行fn，并且让fn中的this变为obj，传递10,20
setTimeout(fn, 1000); //1000ms后执行fn，但是this->window，x/y都是undefined
setTimeout(fn.call(obj, 10, 20), 1000); //这样处理不行，因为在设置定时器的时候，基于call方法，就把fn已经执行了，虽然this和参数都是我们想要的，但是并不是在1000ms后执行的 =>把fn执行的返回结果绑定给定四期，1000ms后执行的是返回结果
解决方案：
//1、包一个函数
setTimeout(function () {
    fn.call(obj, 10, 20);
}, 1000);
//2、使用bind
setTimeout(fn.bind(obj, 10, 20), 1000);
```

### 实现自己的call

如何让obj和fn两个不相干的对象/函数建立联系？

```js
obj.fn = fn;//把fn添加到obj属性上
obj.fn(10, 20);//执行fn
delete obj.fn;//删除Obj上的fn属性，还原原来的样子
```



```js
const fn = function fn(x, y) {
    this.total = x + y;
    return this;
};
let obj = {
    name: 'obj',
    xxx: 10
};
//简单版
Function.prototype.call = function call(context, ...params) {
    // this->fn 当前要处理的函数
    // context->obj 给函数改变的this
    // params->[10,20] 给函数传递的参数

    let result;
    context['xxx'] = this;//让传进来的context新增一个属性xxx,值为当前执行的函数/对象
    result = context['xxx'](...params)//执行context新增的函数，把返回值给到result
    delete context['xxx']//删除该对象上的xxx属性
    return result;//把返回值返回
};
```

**升级版：**

细节点：

1. 如果传入的第一个参数为一个基本数据类型值，那么基本数据类型值设置属性无效，需要把基本数据类型值改为引用数据类型：

   `Object(context)`

   ![image-20200901023841993](https://i.loli.net/2020/09/01/xdko47gyQl6TmEp.png)

2. 如果传入的context是null或者undefined，则设置为window(==双等号的作用是即使传入的是null或者undefined都会相等)

   `context == null ? context = window : null;`

3. 建立关联的属性保持唯一 ，使用Symbol

   `key = Symbol('key');`

```js
//升级版
Function.prototype.call = function call(context, ...params) {
    // this->fn 当前要处理的函数
    // context->obj 给函数改变的this
    // params->[10,20] 给函数传递的参数

    // 细节点：对于context类型的处理（基本数据类型无法设置键值对）
    context == null ? context = window : null;
    !/^(object|function)$/i.test(typeof context) ? context = Object(context) : null;

    // 细节点：建立关联的属性保持唯一 Symbol
    let result,
        key = Symbol('key');
    context[key] = this;
    result = context[key](...params);
    delete context[key];
    return result;
};
```

apply与call的唯一区别就是传入的参数是个数组。

### 实现bind

先来看看事件绑定的不同

```js
const fn = function fn(x, y) {
    this.total = x + y;
    console.log(this);
    return this;
};
window.name = '珠峰';
let obj = {
    name: 'obj'
};

document.body.onclick = fn; //把fn本身绑定给点击事件
document.body.onclick = fn(); //先把fn执行，把执行结果赋值给点击事件
```

```js
document.body.onclick = fn.bind(obj, 10, 20);
// bind原理：执行bind只是先把fn/obj/10/20都存储起来，返回一个匿名函数做我们的事件绑定，当点击的时候，再把匿名函数执行，执行中，把事先存储的那些信息拿过来处理
document.body.onclick = (ev) => {
    fn.call(obj, 10, 20, ev);
};
```

最终方案：

```js
/* 闭包：柯里化函数 */
Function.prototype.bind = function bind(context, ...params) {
    // this->fn 真正要执行的函数
    // context->obj 要给函数改变的this
    // params->[10,20] 要给函数传递的参数
    return (...args) => {
        // args->[ev] 点击行为触发，传递给匿名函数的信息，例如：事件对象    
        this.call(context, ...params.concat(args));
    };
};
```

