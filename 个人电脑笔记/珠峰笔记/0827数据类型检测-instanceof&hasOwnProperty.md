### 通过检测是否属于指定的类来检测数据类型

自定义类：

```js
function Fn(x = 0, y = 0) {
    let str = `珠峰培训`;
    this.total = x + y;
    this.say = function () {
        console.log('OK');
    };
}
//创建两个Fn的实例
let f1 = new Fn(10, 20);
let f2 = new Fn;
```

#### typeof 

> 不能检测具体的数据类型

```js
typeof [] //-->'Object'
typeof /^$/ //-->'Object' 不能细分类型
```



#### instanceof

>  instanceof用来检测某个实例是否属于这个类「也可以基于这种办法检测数据类型：有坑，慎用」

```js
//检测f1是否是Fn的一个实例
console.log(f1 instanceof Fn); // true
console.log(f1 instanceof Array); // false
let arr = [];
console.log(arr instanceof Array); // true
console.log(arr instanceof RegExp); // false
```



#### hasOwnProperty

> 使用Object.prototype上提供的hasOwnProperty方法，来检测某个属性是否为对象的一个**私有属性**

```js
let f1 = new Fn(10, 20);
console.log(f1)
```

控制台输出：

可以看到f1是Fn是t的一个实例，而Fn也是对象类型，所以也是Object的一个实例

![image-20200827181038806](https://i.loli.net/2020/08/27/SVXeghq4MxWrHpv.png)

而Object上有一个hasOwnProperty的方法，可以利用这个方法来检测某个属性是否为对象的一个私有属性

![image-20200827181440474](https://i.loli.net/2020/08/27/ZHSD3go2xOjTIsd.png)

```js
console.log(f1.hasOwnProperty('say')); //true
```



#### in

> 检测当前属性是否属于这个对象「in操作符」：不论是私有还是共有的，只要有检测结果就是true

```js
console.log('say' in f1); //true
console.log('str' in f1); //false
```

#### constructor

> 判断一个已有变量是哪个类的实例

![image-20200828174335382](https://i.loli.net/2020/08/28/ftlxQDuKG6OYIVr.png)