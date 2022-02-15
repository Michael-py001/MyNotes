### 面试题

```js
function Dog(name) {
    this.name = name;
}
Dog.prototype.bark = function () {
    console.log('wangwang');
}
Dog.prototype.sayName = function () {
    console.log('my name is ' + this.name);
}
/*
let sanmao = new Dog('三毛');
sanmao.sayName();
sanmao.bark();
*/
function _new() {
    //=>完成你的代码   
}
let sanmao = _new(Dog, '三毛');
sanmao.bark(); //=>"wangwang"
sanmao.sayName(); //=>"my name is 三毛"
console.log(sanmao instanceof Dog); //=>true
```

### 自己写一个new

#### new一个实例的过程

**new Dog(...)**

1. 创建一个Dog类实例对象

     \+ 对象(空)

     \+ `对象.__proto__`===`Dog.prototype`

2. 把Dog当作普通函数执行:私有上下文/作用域链/形参赋值/变量提升/代码执行...

3. 方法中的this指向创建的实例对象。

4. 看方法返回结果，如果方法中**没有return**或者**返回的是基本类型值**，则返回的都是实例对象，否则以函数返回值为主。也就是说有return且return的是引用数据类型的，则按这个返回，其他都是把obj返回。

自己重写一个new就是把以上步骤一一实现

#### 方案一

> obj.\__proto__ = Func.prototype在IE浏览器下不兼容，所以这个方案有缺陷

```js
// Func[function]：想创造哪个类的实例,就传递这个类
// params[array]：基于剩余运算符获取其余传递的实参，这些值都是给类传递的实参
function _new(Func, ...params) {
    // 创建一个实例对象
    let obj = {};
  	//实例的原型链 = 类的原型
    obj.__proto__ = Func.prototype;

    // 把类当作普通函数执行:基于call方法强制改变上下文中的this为创建的实例对象obj
    let result = Func.call(obj, ...params);

    // 根据返回结果类型，决定返回啥
  	//普通方法：if(result !== null)&&(typeof result === 'object' || typeof result === 'function')) return result
  	//正则判断，result !== null -->是因为null也是一个对象
    if (result !== null && /^(object|function)$/i.test(typeof result)) return result;
    return obj;
} 
let sanmao = _new(Dog, '三毛', 25);
sanmao.bark(); //=>"wangwang"
sanmao.sayName(); //=>"my name is 三毛"
console.log(sanmao instanceof Dog); //=>true
```

#### 方案二---Object.create()

> 使用Object.create()，此方法在IE9-10-11下可以兼容
>
> Object.create([对象A]):创建一个空对象，并且把对象A作为他的原型  空对象.\__proto__===对象A

```js
function _new(Func, ...params) {
    // 优化：不用__proto___（IE不兼容）
  	//let obj = {}
  	//obj.__proto__ = Func.prototype
    let obj = Object.create(Func.prototype);//和以上代码的效果一样
    let result = Func.call(obj, ...params);
    if (result !== null && /^(object|function)$/i.test(typeof result)) return result;
    return obj;
}
```

### 手写实现`Object.create()`

> 思路：new一个类的实例，这个实例的原型链`__proto__`一定是所属类的原型prototype

```js
// Object.create是ES2015（不是ES5，是ES6前身）提供的方法，不兼容低版本浏览器
Object.create = function create(obj) {
    if (obj === null || typeof obj !== "object") {
        throw new TypeError('Object prototype may only be an Object');
    }
		//思路：new一个类的实例，这个实例的原型链__proto__一定是所属类的原型prototype
    function Anonymous() {}//创建一个空的类
    Anonymous.prototype = obj;//把这个类的原型指向传入的obj
    return new Anonymous;//返回一个类的实例
};
```

