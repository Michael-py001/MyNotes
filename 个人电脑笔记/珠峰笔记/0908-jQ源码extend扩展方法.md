



### jQuery.extend扩展自定义方法

> - `$.extend({fn:funcion(){}}) `向对象上扩展方法，完善类库 ->$.fn()
>
> - `$.fn.extend({fn:funcion(){}}))` 向原型上扩展方法，编写JQ插件 ->$(...).fn()

#### 原型上的扩展extend

```js
 jQuery.extend = jQuery.fn.extend = function () {
        var options, name, src, copy, copyIsArray, clone,
            target = arguments[0] || {},
            i = 1,
            length = arguments.length,
            deep = false;

        // 如果第一个值传递的是布尔，让deep等于这个值，第二个值是我们需要扩展的对象
        if (typeof target === "boolean") {
            deep = target;
            target = arguments[i] || {};
            i++;
        }

        // Handle case when target is a string or something (possible in deep copy)
        if (typeof target !== "object" && !isFunction(target)) {
            target = {};
        }

        // Extend jQuery itself if only one argument is passed
        if (i === length) {
            target = this; //this为当前实例
            i--;
        }

        for (; i < length; i++) {

            // Only deal with non-null/undefined values
            if ((options = arguments[i]) != null) {

                // Extend the base object
                for (name in options) {
                    copy = options[name];

                    // Prevent Object.prototype pollution
                    // Prevent never-ending loop
                    if (name === "__proto__" || target === copy) {
                        continue;
                    }

                    // Recurse if we're merging plain objects or arrays
                    if (deep && copy && (jQuery.isPlainObject(copy) ||
                            (copyIsArray = Array.isArray(copy)))) {
                        src = target[name];

                        // Ensure proper type for the source value
                        if (copyIsArray && !Array.isArray(src)) {
                            clone = [];
                        } else if (!copyIsArray && !jQuery.isPlainObject(src)) {
                            clone = {};
                        } else {
                            clone = src;
                        }
                        copyIsArray = false;

                        // Never move original objects, clone them
                        target[name] = jQuery.extend(deep, clone, copy);

                        // Don't bring in undefined values
                    } else if (copy !== undefined) {
                        target[name] = copy;
                    }
                }
            }
        }

        // Return the modified object
        return target;
    };

```

如果我们需要实现默认传一个参数，第一个参数可选时，可以这样实现：

```
//传递的值：数字、函数 或者 函数（数字不传递默认是0）
/* function fn() {
    var x = 0,
        y = arguments[0];//获取第一个实参
    if (typeof y === "number") { //如果是数字
        x = y; //x替换y
        y = arguments[1]; //y变成第二个参数
    }
    console.log(x, y);
}
fn(0, function () {});
fn(function () {});
```

#### 工具类上的extend方法

```js
   jQuery.extend({
        // 遍历数组/类数组/对象
        each: function (obj, callback) {
            var length, i = 0;
            if (isArrayLike(obj)) {
                // 数组或者类数组基于for循环完成
                length = obj.length;
                for (; i < length; i++) {
                    var item = obj[i];
                    var res = callback.call(item, i, item);
                    //在内置forEach的基础上可以控制循环结束：只要回调函数中返回false，循环就可以结束了
                    if (res === false) {
                        break;
                    }
                }
            } else {
                // 如果是对象，基于for in循环
                for (i in obj) {
                    if (callback.call(obj[i], i, obj[i]) === false) {
                        break;
                    }
                }
            }
            return obj;
        }
    });
```

### jQ的each方法

#### jQ对象内的each:

```js
jQuery.extend({
        // 遍历数组/类数组/对象
        each: function (obj, callback) {
            var length, i = 0;
            if (isArrayLike(obj)) {
                // 数组或者类数组基于for循环完成
                length = obj.length;
                for (; i < length; i++) {
                    var item = obj[i];
                  	//实现循环中断
                    var res = callback.call(item, i, item);
                    //在内置forEach的基础上可以控制循环结束：只要回调函数中返回false，循环就可以结束了
                    if (res === false) {
                        break;
                    }
                }
            } else {
                // 如果是对象，基于for in循环
                for (i in obj) {
                    if (callback.call(obj[i], i, obj[i]) === false) {
                        break;
                    }
                }
            }
            return obj;
        }
    });
```



#### 原型上的each方法:

```js
jQuery.fn = jQuery.prototype = {
        constructor: jQuery, //原型重定向后constructor会丢失，需要手动设置
        jquery: version,
        // 原型上的each是调用工具类(jQ对象上)的each，
        each: function (callback) {
            // 调用原型上的each方法：核心本质也是调用工具类的each方法
            return jQuery.each(this, callback);
        },
        sort: [].sort
 }
```

### for...of循环

> 可以被迭代的数据结构（拥有Symbol.iterator这个属性的即可被迭代 ），可以基于for of循环处理

哪些属性有**Symbol.iterator**这个属性呢？

+ **数组:** `Array.prototype[Symbol.iterator] = function values() { [native code] }`**在Array.prototype上有这个属性**：

  ![image-20200908155618255](https://i.loli.net/2020/09/08/BG2qwV3LdeHhmkK.png)

+ **arguments:**  `arguments[Symbol.iterator] = function values() { [native code] }`**在Arguments上也有**：

  ![image-20200908160210962](https://i.loli.net/2020/09/08/9fHBxytESi4JFZa.png)

+ NodeList节点类数组集合
  ![image-20200908160416595](https://i.loli.net/2020/09/08/J7kzFYi69MqryTa.png)

+ HTMLCollection元素集合
  ![image-20200908160526311](https://i.loli.net/2020/09/08/XCkoALRSlugbny6.png)

+ new Set()
  ![image-20200908160612479](https://i.loli.net/2020/09/08/YSkU3tfdr9pLVxv.png)

+ new Map()
  ![image-20200908160651105](https://i.loli.net/2020/09/08/g6rcTI8nfZwQdiY.png)

+ ...

+ 普通对象默认是不具备可迭代性的，因为它不具备 Symbol.iterator 这个属性

### 如何让对象可被迭代呢？

> 普通对象默认是不具备可迭代性的，因为它不具备 Symbol.iterator 这个属性；
>
> 只需要设置Symbol.iterator这个属性的即可被迭代

```js
let arr = {
    0: '123',
    1: '234',
    2: '345',
    length: 4,
    age: 18,
    [Symbol.iterator]: Array.prototype[Symbol.iterator]
}
for (key of arr) {
    console.log(key);
}
```

在jQ源码上：让JQ集合(类数组)变为可被迭代

```js
// 让JQ集合(类数组)变为可别迭代的，这样可以基于for of循环遍历这个集合
    if (typeof Symbol === "function") {
        jQuery.fn[Symbol.iterator] = arr[Symbol.iterator];
    }
```



```js
 //jQ源码上的sort方法也是调用内置的sort方法
 sort: [].sort
// $(...).sort(function(){}) //类数组设置了Symbol.iterator，可以被迭代
// sort: [].sort--> [].sort.call($(...),function(){}) //相当于内置的sort把this改成类数组，就可以借用数组上的方法了
```

