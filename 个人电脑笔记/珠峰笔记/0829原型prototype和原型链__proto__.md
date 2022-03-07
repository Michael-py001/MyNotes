### 一道试题入手

```js
function Fn() {
    this.x = 100;
    this.y = 200;
    this.getX = function () {
        console.log(this.x);
    }
}
Fn.prototype.getX = function () {//给原型上扩展方法
    console.log(this.x);
};
Fn.prototype.getY = function () {
    console.log(this.y);
};
let f1 = new Fn;
let f2 = new Fn;
console.log(f1.getX === f2.getX);
console.log(f1.getY === f2.getY);
console.log(f1.__proto__.getY === Fn.prototype.getY);
console.log(f1.__proto__.getX === f2.getX);
console.log(f1.getX === Fn.prototype.getX);
console.log(f1.constructor);
console.log(Fn.prototype.__proto__.constructor);
f1.getX();
f1.__proto__.getX();
f2.getY();
Fn.prototype.getY();
```

### 原型`prototype`和原型链`__proto__`

#### `prototype  原型`

1. 所有的类都是函数数据类型(包括内置类)

2. 所有函数都天生自带一个属性：**`prototype `**

3. prototype是一个对象数据类型，上面存储了能供实例调用的公共属性和方法

4. 并且在类的原型prototype上，默认有一个属性：**constructor--构造函数**，属性值是当前类本身

   ![image-20200830004521409](https://i.loli.net/2020/08/30/d6QGv2HOPrt1DIg.png)

#### `__proto__  原型链`

1. 所有对象数据类型也天生自带一个属性：`__proto__ `

2. `__proto__`的属性值：当前这个实例所属类的prototype，也就是他上一级的原型prototype
   `实例.__proto__`===`类.prototype`

#### 都有哪些值是对象数据类型的？

1. 普通对象/数组对象/正则对象/日期对象/类数组对象/DOM对象...
2. 大部分的实例(除了基本数据类型外)都是对象类型
3. 类的原型-->`prototype`也是对象类型
4. 函数也是对象--->'函数的三种角色'
5. .....[万物皆对象]，所有的值都是对象类型

#### Object内置对象类

- Object是所有对象数据类型的‘基类’，`Object.prototye.__proto__`指向是就是`Object.prototype`，也就是指向自己，这样没有意义，所以`Object.prototye.__proto__`等于`null`

- 在不知道new谁出来的对象下，所有的对象类型值都是Object内置类的一个实例

#### 原型链查找机制

f1.getX

- 首先看是否为自己的私有属性，如果是私有的，那么操作的也就是私有的
- 如果不是私有属性，则会基于`__proto__`原型链查找所属类原型上的公共属性和方法
- 如果所属类的原型上也没有，则继续基于`__proto__`向上查找，直到找到`Object.prototype`为止
  我们把这种查找机制成为--[原型链查找机制]

#### 属性的共有和私有

getX,getY是Fn上`prototype`的属性方法，对于`Fn.prototype`来说是私有的，但是对于Fn创建出来的实例f1/f2来说是公有的。