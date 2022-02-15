[TOC]



# Vue第二天-响应式原理&自定义指令&watch&computed

## Vue2.0的响应式原理

> Vue2.0中的两个缺陷：
>
> ​	1）缺陷一：Vue对新增加的对象属性不会被监控到，使用vm.$set；
>
> ​	2) 缺陷二：1.数组中不能通过索引直接改值 2.不能通过arr.length修改数组长度
>
> 在Vue3.0中使用Proxy代理的方式进行了完善。

Vue对通过数组索引直接修改属性，或者对数组长度进行修改，是监测不到的，所以要包裹一个方法，把这个数据添加到响应式系统中。

```js
this.$set(你要修改的数组, 要修改的那个索引值, 新的值)
```

### 模拟对象与数组的响应式

> 1. `Object.defineProperty()`的文档：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty
>
>      obj:需要定义属性的对象
>
>      key:需要修改或定义的属性名称
>
>      value:要定义或修改的属性描述符。(对象)
>
> 2. `Object.create()`:创建一个新对象，使用现有的对象来提供新创建的对象的__proto__。返回一个新对象，带着指定的原型对象和属性。
>    let myProto = Object.create(arrProto) //相当于myProto.__proto__ = arrProto
> 3. 重点：
>    1. 对嵌套的对象再次用observer( )方法递归处理
>    2. 封装$set( )方法
>    3. 重新包裹`Array.prototype`中的部分方法-->寄生式组合继承，调用时可以渲染视图。

```js
// VUe2.0响应式原理

let render = function () {
    console.log("视图渲染了")
}
//对象
let obj = {
    name: "li",
    age: 18,
    todo: {
        first: "shopping",
        second: "lunch"
    }
}
//数组
let arr = [1, 2, 3]
// 1）缺陷一：Vue对新增加的对象属性不会被监控到，使用vm.$set

// 2) 缺陷二：1.数组中不能通过索引直接改值  2.不能通过arr.length修改数组长度
//    Vue中对push pop shift unshift splice sort reverse这些方法进行了包裹，在Vue中使用这些方法时是响应式的，所以我们这里也要模拟包裹这些方法

let methods = ["push", "pop", "shift", "unshift", "sort", "splice", "reverse"];

let arrProto = Array.prototype;
//寄生式组合继承
// Object.create()方法创建一个新对象，使用现有的对象来提供新创建的对象的__proto__。返回一个新对象，带着指定的原型对象和属性。
let myProto = Object.create(arrProto) //相当于myProto.__proto__ = arrProto
// 重写methods方法
methods.forEach(method => {
    myProto[method] = function () {
        render();
        arrProto[method].call(this, ...arguments);
    }
})


// Vue对通过数组索引直接修改属性，或者对数组长度进行修改，是监测不到的，所以要包裹一个方法，把这个数据添加到响应式系统中
let $set = function (data, key, value) {
    if (Array.isArray(data)) {
      //$set中使用的是splice方法
      //this.$set(你要修改的数组, 要修改的那个索引值, 新的值) 这里为了解决通过索引修改数组值时的响应式
        return data.splice(key, 1, value)
    }
    defineReactive(data, key, value)
}
// 对一个对象进行监听
let observer = function (obj) {
    // 如果是数组，直接返回(Vue中不允许直接通过索引修改值)
    if (Array.isArray(obj)) {
        // 把传进来的arr继承自己封装的数组方法
        obj.__proto__ = myProto;
        return
    }
    if (typeof obj == "object") {
        for (let key in obj) {
            defineReactive(obj, key, obj[key])
        }
    }
}

// 拦截对象的取值和修改进行处理
let defineReactive = function (obj, key, value) {
    // 对obj中嵌套的对象进行递归处理
    observer(value)

    // Object.defineProperty()的文档：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty
    //obj:需要定义属性的对象
    //key:需要修改或定义的属性名称
    //value:要定义或修改的属性描述符。(对象)
    Object.defineProperty(obj, key, {
        get() {
            return value
        },
        set(newValue) {
            // 如果新旧值不同，则进行处理
            if (newValue !== value) {
                observer(newValue) //对新设置的对象也进行监听

                render();//在设置新的值的同时，视图重新渲染一次
                value = newValue;
            }
        }
    })
}


// 开始监视obj/arr的修改
observer(arr)

// obj.name = "lili";//会触发render函数


/* obj.age = {
    age: 10,
    sex: "man"
}
//会触发两次视图渲染
obj.age.age = 22 */

// 通过$set把新属性添加到响应式系统中
// $set(obj, "student", "ali")
// obj.student = "alia"




// arr[0] = 100
// arr.push(99) //使用的是自己包裹过的数组方法，会渲染视图
$set(arr, 0, 100)
```

## Vue3.0响应式原理

> Vue3.0中使用proxy代理的方式实现对数据的修改和取值的监控。
> proxy文档：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy

重点：

1. 使用`Reflect.set/get`，可以不用区分数组和对象。
 https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect/get
2. 处理数组时，排除数组中的length属性。

## Proxy

> **Proxy** 对象用于定义基本操作的自定义行为（如属性查找、赋值、枚举、函数调用等）

语法：

```js
const p = new Proxy(target, handler)
```

参数：

- `target`

  要使用 `Proxy` 包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）。

- `handler`

  一个通常以函数作为属性的对象，各属性中的函数分别定义了在执行各种操作时代理 `p` 的行为。

### [`handler.get()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/get)

​		在读取代理对象的某个属性时触发该操作，比如在执行 `proxy.foo` 时。

​		语法

```js
var p = new Proxy(target, {
  get: function(target, property, receiver) {
  }
});
```

**参数**

以下是传递给get方法的参数，`this上下文绑定在`handler对象上.

- `target`

  目标对象。

- `property`

  被获取的属性名。

- `receiver`

  Proxy或者继承Proxy的对象

**返回值**

get方法可以返回任何值。

**描述**

**`handler.get`** 方法用于拦截对象的读取属性操作。

**拦截**

该方法会拦截目标对象的以下操作:

- 访问属性: `proxy[foo]和` `proxy.bar`
- 访问原型链上的属性: `Object.create(proxy)[foo]`
- [`Reflect.get()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect/get)

### [`handler.set()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/set)

在给代理对象的某个属性赋值时触发该操作，比如在执行 `proxy.foo = 1` 时

**语法**

```
const p = new Proxy(target, {
  set: function(target, property, value, receiver) {
  }
});
```

参数

以下是传递给 `set()` 方法的参数。`this` 绑定在 handler 对象上。

- `target`

  目标对象。

- `property`

  将被设置的属性名或 [`Symbol`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol)。

- `value`

  新属性值。

- `receiver`

  最初被调用的对象。通常是 proxy 本身，但 handler 的 set 方法也有可能在原型链上，或以其他方式被间接地调用（因此不一定是 proxy 本身）。**比如：**假设有一段代码执行 `obj.name = "jen"`， `obj` 不是一个 proxy，且自身不含 `name` 属性，但是它的原型链上有一个 proxy，那么，那个 proxy 的 `set()` 处理器会被调用，而此时，`obj` 会作为 receiver 参数传进来。

**返回值**

`set()` 方法应当返回一个布尔值。

- 返回 `true` 代表属性设置成功。
- 在严格模式下，如果 `set()` 方法返回 `false`，那么会抛出一个 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError) 异常。

**描述**

`handler.set()` 方法用于拦截设置属性值的操作。

**拦截**

该方法会拦截目标对象的以下操作:

- 指定属性值：`proxy[foo] = bar` 和 `proxy.foo = bar`
- 指定继承者的属性值：`Object.create(proxy)[foo] = bar`
- [`Reflect.set()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect/set)

## Reflect

> **Reflect** 是一个内置的对象，它提供拦截 JavaScript 操作的方法。这些方法与[proxy handlers](https://wiki.developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler)的方法相同。

`Reflect.get(target, propertyKey[, receiver\])`

​		`Reflect.get()`方法与从 对象 (`target[propertyKey]`) 中读取属性类似，但它是通过一个函数执行来操作的。

`Reflect.set(target, propertyKey, value[, receiver\])`

​		将值分配给属性的函数。返回一个[`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean)，如果更新成功，则返回`true`。

### 响应式原理实现

```js
//3.0中支持数组和对象
let obj = {
    name: "gg",
    location: {
        x: 100,
        y: 200
    },
    arr: [1, 2, 3]
}
let render = function () {
    console.log("视图渲染了");
}
let handler = {
    get(obj, prop) {
        // 如果obj中的一个属性值是个对象，则再次调用handler递归处理
        if (typeof obj[prop] == "object" && obj[prop] !== null) {
            return new Proxy(obj[prop], handler)
        }
        return Reflect.get(obj, prop)  //获取值
        /* 
        Reflect.get方法允许你从一个对象(对象和属性都可以)中取属性值。就如同属性访问器 语法，但却是通过函数调用来实现(不是对象时有异常报错)
        在Vue3.0中使用Reflect.get获取值，就不用区分数组和对象了，解决了2.0中的两个问题
        */
    },
    set(obj, key, value) {
        if (key == "length") return true //不能只写return handler中必须要有一个返回值，这里写true就行了
        render() // 设置值时视图渲染
        return Reflect.set(obj, key, value) //设置值
    }
}

let proxy = new Proxy(obj, handler)

// 修改和获取时，通过proxy进行操作
proxy.location.x = "234"  //设置时包含获取，在get里进行递归了一次，视图会更新
console.log(obj.arr);

proxy.arr.push(100) //proxy处理数组时，会对每一项都进行代理，包括数组中的length属性，这里需要排除length
console.log(obj.arr);

proxy.arr[1] = 999 //通过proxy，可以实现通过数组索引修改值
console.log(obj.arr);

proxy.student = "alili";//对象也可以直接新增属性了
console.log(obj);
```

## 自定义指令

> 自定义指令文档：
>
> https://cn.vuejs.org/v2/api/#Vue-directive
>
> https://cn.vuejs.org/v2/guide/custom-directive.html
>
> 

### 全局指令Vue.directive

> 参数：
>
>  - {string} id  指令名称 v-xxx的部分
>  - {Function | Object} [definition]  函数，
>     - 可以直接写function，这样将会被 `bind` 和 `update` 钩子函数调用，
>     - 也可以一个个钩子函数分开写：
>
> **用法**：
>
> 注册或获取全局指令。
>
> - 分离式写法：
>
>   ```js
>   // 注册
>   Vue.directive('my-directive', {
>     bind: function () {},
>     inserted: function () {},
>     update: function () {},
>     componentUpdated: function () {},
>     unbind: function () {}
>   })
>   ```
>
> - 函数式写法
>
>   ```js
>   // 注册 (指令函数)
>   Vue.directive('my-directive', function () {
>     // 这里将会被 `bind` 和 `update` 调用
>   })
>   ```
>
> 返回值：
>
> ```js
> // getter，返回已注册的指令
> var myDirective = Vue.directive('my-directive')
> ```
>
> 

#### **标签中的指令：**

- `v-color="red"` 这样双引号包裹代表red是一个**变量**，需要在data里声明后才能用，

- `v-color="'red'"` 再使用单引号包裹，就代表是一个字符串，可直接使用。

效果：设置标签的color属性。

```html
//html
 <div id="app">
        <!-- v-color="red" 这样双引号包裹代表red是一个变量，需要在data里声明后才能用，v-color="'red'" 再使用单引号包裹，就代表是一个字符串 -->
        <div v-color="'red'">自定义颜色指令</div>
    </div>
```



```js
//js
Vue.directive('color', function (el, bindings, vnode) {
  //el 绑定的元素
  //binddings 指令的参数
  //vnode Vue 编译生成的虚拟节点 vnode.context 代表当前vue实例->vm
  // 设置静态值
  // el.style.background = "red"
  // 动态获取参数值
  el.style.background = bindings.value
})
let vm = new Vue({
  el: "#app",
  data: {
    name: "li",
    friends: {
      name: "ti",
      age: 19
    }
  }
})
```

#### 钩子函数

一个指令定义对象可以提供如下几个钩子函数 (均为可选)：

- `bind`：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
- `inserted`：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
- `update`：所在组件的 VNode 更新时调用，**但是可能发生在其子 VNode 更新之前**。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新 (详细的钩子函数参数见下)。

我们会在[稍后](https://cn.vuejs.org/v2/guide/render-function.html#虚拟-DOM)讨论[渲染函数](https://cn.vuejs.org/v2/guide/render-function.html)时介绍更多 VNodes 的细节。

- `componentUpdated`：指令所在组件的 VNode **及其子 VNode** 全部更新后调用。
- `unbind`：只调用一次，指令与元素解绑时调用。

#### 钩子函数中的参数

> 钩子函数的参数 (即 `el`、`binding`、`vnode` 和 `oldVnode`)

指令钩子函数会被传入以下参数：

- `el`：指令所绑定的元素，可以用来直接操作 DOM。

- ```
  binding
  ```

  ：一个对象，包含以下 property：

  - `name`：指令名，不包括 `v-` 前缀。
  - `value`：指令的绑定值，例如：`v-my-directive="1 + 1"` 中，绑定值为 `2`。
  - `oldValue`：指令绑定的前一个值，仅在 `update` 和 `componentUpdated` 钩子中可用。无论值是否改变都可用。
  - `expression`：字符串形式的指令表达式。例如 `v-my-directive="1 + 1"` 中，表达式为 `"1 + 1"`。
  - `arg`：传给指令的参数，可选。例如 `v-my-directive:foo` 中，参数为 `"foo"`。
  - `modifiers`：一个包含修饰符的对象。例如：`v-my-directive.foo.bar` 中，修饰符对象为 `{ foo: true, bar: true }`。

- `vnode`：Vue 编译生成的虚拟节点。移步 [VNode API](https://cn.vuejs.org/v2/api/#VNode-接口) 来了解更多详情。

- `oldVnode`：上一个虚拟节点，仅在 `update` 和 `componentUpdated` 钩子中可用。

**注意：**除了 `el` 之外，其它参数都应该是只读的，切勿进行修改。如果需要在钩子之间共享数据，建议通过元素的 [`dataset`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLElement/dataset) 来进行。

### 局部指令

> 直接在实例vm里面写`directives`，
>
> 可以有分离式和函数式写法：
>
> - focus:{bind(){..}, xxx(){...}}
> - focus(el,bindinds,vnode){ ... }

#### 案例一：

> 指令：`v-focus`
>
> 实现的效果：进入页面后自动聚焦到input框

```html
//html
<input type="text" v-focus>
```



```js
 let vm = new Vue({
            el: "#app",
            data: {
                msg: "hello"
            },
            // 局部指令
            directives: {
                // 指令的名称 focus
                focus: {
                    bind(el) {//只调用一次，指令第一次绑定到元素时调用
                        console.log("el绑定到页面时执行");
                    },
                    inserted(el) {//el被绑定元素插入父节点时调用
                        console.log(el);
                        el.focus()//让光标聚焦到input框
                    },
                    update() {//所在组件的 VNode 更新时调用

                    },
                    unbind() {//只调用一次，指令与元素解绑时调用。

                    }
                },
                //函数式写法
                // focus(el, binddings, vnode) {

                // }
            }
        })
```

#### 案例二：

> 指令：`v-click-outside`
>
> 实现的效果：点击Input框，显示选择日历，光标离开聚焦时，隐藏选择日历。

html:

```html
 <span>点击选择日期</span>
<div v-click-outside="hide" class="wraper"
     <!-- @focus 聚焦事件 -->>
  <input type="text" @focus="show">
  <div class="content" v-if="isShow">
    <label for="date">选择日期</label><input type="date" name="date" id="">
  </div>
</div>
```

js:

```js
 let vm = new Vue({
            el: "#app",
            data: {
                msg: "hello",
                isShow: false
            },
            methods: {
                show() {
                    this.isShow = true
                },
                hide() {
                    this.isShow = false
                }
            },
            // 局部指令
            directives: {
                //clickOutside
                "click-outside": {
                    bind(el, binddings, vnode) {//只调用一次，指令第一次绑定到元素时调用
                        function hide(e) {
                            // 使用contains判断是一个元素否包含元素
                            if (!el.contains(e.target)) {
                                // 如果不包含则隐藏
                                console.log("隐藏");
                                vnode.context.hide()
                            }
                        }
                        document.addEventListener("click", hide)
                    },
                    inserted(el) {//el被绑定元素插入父节点时调用

                    },
                    update() {//所在组件的 VNode 更新时调用

                    },
                    unbind() {//只调用一次，指令与元素解绑时调用。
                        console.log("解绑");
                    }
                },
            }
        })
```

效果：

![image-20201019030659126](https://i.loli.net/2020/10/19/zTAZrUbh1Hj2VvY.png)

实现动态传参，运行不同的函数：

> `bindings.expression`这个属性就包含了传进来的参数：hide，
>
> 然后动态运行这个函数：
>
> ```js
> vnode.context[bindings.expression]()
> ```
>
> 

```js
"click-outside": {
  bind(el, bindings, vnode) {
    //bindings.expression这个属性就包含了传进来的参数：hide
    console.log(bindings);
    function hide(e) {
      // 使用contains判断是一个元素否包含元素
      if (!el.contains(e.target)) {
        // 如果不包含则隐藏
        console.log("隐藏");
        // vnode.context.hide()
        //动态参数
        vnode.context[bindings.expression]()
      }
    }
    //监听click事件
    document.addEventListener("click", hide)
  },
}
```

### 指令的销毁

> 自定义组件中`unbind()`是什么时候会触发呢？

只有在使用Vue中的方法去移除DOM时才会销毁，比如设置一个开关:`v-if`，点击时变成`false`，这时就会触发`unbind()`，解绑指令。

使用原生的方法，比如`node.remove()`，DOM是被移除了，但是和Vue没有关联，不会触发`unbind()`

#### 使用`v-if`销毁DOM：

结构：

```html
<span>点击选择日期</span>
<div v-click-outside="hide" class="wraper" v-if="isOff">
  <!-- @focus 聚焦事件 -->
  <input type="text" @focus="show">
  <div class="content" v-if="isShow">
    <label for="date">选择日期</label><input type="date" name="date" id="">
  </div>
</div>
<button @click="off">指令销毁</button>
```

​		在点击【指令销毁】时，会把`isOff`改为`false`，这时会通过Vue把DOM销毁，这时就会触发解绑`unbind()`

![image-20201019030825398](https://i.loli.net/2020/10/19/wW6HO3yoMabtFdr.png)

#### 使用原生方法移除DOM：

![image-20201019030619487](https://i.loli.net/2020/10/19/BOiq4fbmVRc9sN8.png)

### 函数简写

在很多时候，你可能想在 `bind` 和 `update` 时触发相同行为，而不关心其它的钩子。比如这样写：

```js
Vue.directive('color-swatch', function (el, binding) {
  el.style.backgroundColor = binding.value
})
```

## watch

### 选项-watch

- **类型**：`{ [key: string]: string | Function | Object | Array }`

- **详细**：

  一个对象，键是需要观察的表达式，值是对应回调函数。值也可以是方法名，或者包含选项的对象。Vue 实例将会在实例化时调用 `$watch()`，遍历 watch 对象的每一个 property。

- **选项：**

  - **deep**
    为了发现对象内部值的变化，可以在选项参数中指定 `deep: true`
  - **immediate**
    在选项参数中指定 `immediate: true` 将立即以表达式的当前值触发回调：

  使用选项时，请用对象的方式，如:

  ​	注意函数前需要写handler

  ```js
  msg: {
        handler: function (val, oldVal) { /* ... */ },
        deep: true
      },
  ```

  如果不需要选项，则直接写函数：

  ```js
  msg: function (val, oldVal) {
        console.log('new: %s, old: %s', val, oldVal)
      },
  ```

  

- **示例**：

  ```js
  var vm = new Vue({
    data: {
      a: 1,
      msg:"hello,"
      b: 2,
      c: 3,
      d: 4,
      e: {
        f: {
          g: 5
        }
      }
    },
    watch: {
      a: function (val, oldVal) {
        console.log('new: %s, old: %s', val, oldVal)
      },
      //直接按名字写函数也行
      //相当于直接调用handler函数
      msg(newVal, oldVal) {
                      console.log('new: %s, old: %s', newVal, oldVal);
                  },
      // 方法名
      b: 'someMethod',
      
      // 该回调会在任何被侦听的对象的 property 改变时被调用，不论其被嵌套多深
      c: {
        handler: function (val, oldVal) { /* ... */ },
        deep: true
      },
      
      // 该回调将会在侦听开始之后被立即调用
      d: {
        handler: 'someMethod',
        immediate: true
      },
      
      // 你可以传入回调数组，它们会被逐一调用
      e: [
        'handle1',
        function handle2 (val, oldVal) { /* ... */ },
        {
          handler: function handle3 (val, oldVal) { /* ... */ },
          /* ... */
        }
      ],
      
      // watch vm.e.f's value: {g: 5}
      'e.f': function (val, oldVal) { /* ... */ }
    }
  })
  vm.a = 2 // => new: 2, old: 1
  ```

  注意，**不应该使用箭头函数来定义 watcher 函数**

### 实例-watch

### vm.$watch( expOrFn, callback, options] )

- **参数**：
  - `{string | Function} expOrFn`
  - `{Function | Object} callback`
  - `{Object} [options]`
    - `{boolean} deep`
    - `{boolean} immediate`
- **返回值**：`{Function} unwatch`

- **用法**：

  观察 Vue 实例上的一个表达式或者一个函数计算结果的变化。回调函数得到的参数为新值和旧值。表达式只接受简单的键路径。对于更复杂的表达式，用一个**函数**取代。

  注意：在变更 (不是替换) 对象或数组时，旧值将与新值相同，因为它们的引用指向同一个对象/数组。Vue 不会保留变更之前值的副本。

- **与选项中的watch的区别**：
    函数前不用写`handler`

- **示例**：

  - 键路径

    ```js
    // 键路径
    vm.$watch('a.b.c', function (newVal, oldVal) {
      // 做点什么
    })
    ```

  - 函数

    ```js
    // 函数
    data: {
      a: 10,
      b: 20
    },
    vm.$watch(
      function () {
        // 表达式 `this.a + this.b` 每次得出一个不同的结果时
        // 处理函数都会被调用。
        // 这就像监听一个未被定义的计算属性
        return this.a + this.b
      },
      function (newVal, oldVal) {
        // 做点什么
        console.log('函数:new: %s, old: %s', newVal, oldVal);
      }
    )
    ```

    当data中的a,b数据发生变化时，会触发函数，并且是`this.a + this.b`的值。

    ![image-20201019142714061](https://i.loli.net/2020/10/19/2Qa7CfnIvVlK5wc.png)

  `vm.$watch` 返回一个取消观察函数，用来停止触发回调：

  ```js
  var unwatch = vm.$watch('a', cb)
  // 之后取消观察
  unwatch()
  ```

- **选项：deep**

  为了发现对象内部值的变化，可以在选项参数中指定 `deep: true`。注意监听数组的变更不需要这么做。

  ```
  vm.$watch('someObject', callback, {
    deep: true
  })
  vm.someObject.nestedValue = 123
  // callback is fired
  ```

- **选项：immediate**

  在选项参数中指定 `immediate: true` 将立即以表达式的当前值触发回调：

  ```
  vm.$watch('a', callback, {
    immediate: true
  })
  // 立即以 `a` 的当前值触发回调
  ```

  注意在带有 `immediate` 选项时，你不能在第一次回调时取消侦听给定的 property。

  ```js
  // 这会导致报错
  var unwatch = vm.$watch(
    'value',
    function () {
      doSomething()
      unwatch()
    },
    { immediate: true }
  )
  ```

  如果你仍然希望在回调内部调用一个取消侦听的函数，你应该先检查其函数的可用性：

  ```js
  var unwatch = vm.$watch(
    'value',
    function () {
      doSomething()
      if (unwatch) {
        unwatch()
      }
    },
    { immediate: true }
  )
  ```

## computed

> 计算属性不能与data中的属性重名。

- **类型**：`{ [key: string]: Function | { get: Function, set: Function } }`

- **详细**：

  计算属性将被混入到 Vue 实例中。所有 getter 和 setter 的 this 上下文自动地绑定为 Vue 实例。

  注意如果你为一个计算属性使用了箭头函数，则 `this` 不会指向这个组件的实例，不过你仍然可以将其实例作为函数的第一个参数来访问。

  ```js
  computed: {
    aDouble: vm => vm.a * 2
  }
  ```

  计算属性的结果会被缓存，除非依赖的响应式 property 变化才会重新计算。注意，如果某个依赖 (比如非响应式 property) 在该实例范畴之外，则计算属性是**不会**被更新的。

模板内的表达式非常便利，但是设计它们的初衷是用于简单运算的。在模板中放入太多的逻辑会让模板过重且难以维护。例如：

```html
<div id="example">
  {{ message.split('').reverse().join('') }}
</div>
```

在这个地方，模板不再是简单的声明式逻辑。你必须看一段时间才能意识到，这里是想要显示变量 `message` 的翻转字符串。当你想要在模板中多包含此处的翻转字符串时，就会更加难以处理。

所以，对于任何复杂逻辑，你都应当使用**计算属性**。

#### 基础例子

```js
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
```

结果：

Original message: "Hello"

Computed reversed message: "olleH"

这里我们声明了一个计算属性 `reversedMessage`。我们提供的函数将用作 property `vm.reversedMessage` 的 getter 函数：
也就是说获取`vm.reversedMessage`时，会自动调用这个getter函数。

```js
console.log(vm.reversedMessage) // => 'olleH'
vm.message = 'Goodbye'
console.log(vm.reversedMessage) // => 'eybdooG'
```

你可以打开浏览器的控制台，自行修改例子中的 vm。`vm.reversedMessage` 的值始终取决于 `vm.message` 的值。

你可以像绑定普通 property 一样在模板中绑定计算属性。Vue 知道 `vm.reversedMessage` 依赖于 `vm.message`，因此当 `vm.message` 发生改变时，所有依赖 `vm.reversedMessage` 的绑定也会更新。而且最妙的是我们已经以声明的方式创建了这种依赖关系：计算属性的 getter 函数是没有副作用 (side effect) 的，这使它更易于测试和理解。

#### 计算属性缓存 vs 方法

你可能已经注意到我们可以通过在表达式中调用方法来达到同样的效果：

```js
<p>Reversed message: "{{ reversedMessage() }}"</p>
// 在组件中
methods: {
  reversedMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```

我们可以将同一函数定义为一个方法而不是一个计算属性。两种方式的最终结果确实是完全相同的。然而，不同的是**计算属性是基于它们的响应式依赖进行缓存的**。只在相关响应式依赖发生改变时它们才会重新求值。这就意味着只要 `message` 还没有发生改变，多次访问 `reversedMessage` 计算属性会立即返回之前的计算结果，而不必再次执行函数。

这也同样意味着下面的计算属性将不再更新，因为 `Date.now()` 不是响应式依赖：

```js
computed: {
  now: function () {
    return Date.now()
  }
}
```

#### 计算属性 vs 侦听属性

Vue 提供了一种更通用的方式来观察和响应 Vue 实例上的数据变动：**侦听属性**。当你有一些数据需要随着其它数据变动而变动时，你很容易滥用 `watch`——特别是如果你之前使用过 AngularJS。然而，通常更好的做法是使用计算属性而不是命令式的 `watch` 回调。细想一下这个例子：

```js
<div id="demo">{{ fullName }}</div>
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar'
  },
  watch: {
    firstName: function (val) {
      this.fullName = val + ' ' + this.lastName
    },
    lastName: function (val) {
      this.fullName = this.firstName + ' ' + val
    }
  }
})
```

上面代码是命令式且重复的。将它与计算属性的版本进行比较：

```js
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: {
    //当firstName或者lastName的值发生改变时，触发fullName函数
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})
```

好得多了，不是吗？

#### 计算属性的 setter

计算属性默认只有 getter，不过在需要时你也可以提供一个 setter：

```js
// ...
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
// ...
```

现在再运行 `vm.fullName = 'John Doe'` 时，setter 会被调用，`vm.firstName` 和 `vm.lastName` 也会相应地被更新。

#### 输入文字后发起请求的案例

> - 使用lodash库的debounce防抖，capitalize首字母大写；
>
> - 使用Axios库发起请求
> - 在生命周期的`created`阶段，创建一个`getAnswer`的防抖函数。
> - 使用`v-model`监听question响应式数据，当输入内容时,question会发生改变
> - 在`watch`选项中监听question的值变化，一单发生变化，则更改answer内容，以及调用`this.debouncedGetAnswer()`
> - 防抖函数默认会在输入结束后，调用`getAnswer`函数，继而发起请求。

效果：

![rotateXYZ](https://i.loli.net/2020/10/19/9mhTcK7UDYfiR6a.gif)

```vue
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>watch</title>
</head>

<body>
    <div id="watch-example">
        <p>
            Ask a yes/no question:
            <input v-model="question">
        </p>
        <p>{{ answer }}</p>
    </div>
    <script src="./vue.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/axios@0.12.0/dist/axios.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/lodash@4.13.1/lodash.min.js"></script>
 <script>
        var watchExampleVM = new Vue({
            el: '#watch-example',
            data: {
                question: '',
                answer: 'I cannot give you an answer until you ask a question!'
            },
            watch: {
                // 如果 `question` 发生改变，这个函数就会运行
                question: function (newQuestion, oldQuestion) {
                    this.answer = 'Waiting for you to stop typing...'
                    this.debouncedGetAnswer()
                }
            },
            created: function () {
                // `_.debounce` 是一个通过 Lodash 限制操作频率的函数。
                // 在这个例子中，我们希望限制访问 yesno.wtf/api 的频率
                // AJAX 请求直到用户输入完毕才会发出。想要了解更多关于
                // `_.debounce` 函数 (及其近亲 `_.throttle`) 的知识，
                // 请参考：https://lodash.com/docs#debounce
                this.debouncedGetAnswer = _.debounce(this.getAnswer, 500, {
                    'leading': false, //[默认值false]在输入结束后才发起请求
                    // 'trailing': false
                })
            },
            methods: {
                getAnswer: function () {
                    console.log("ok");
                    if (this.question.indexOf('?') === -1) {
                        this.answer = 'Questions usually contain a question mark. ;-)'
                        return
                    }
                    this.answer = 'Thinking...'
                    var vm = this
                    axios.get('https://yesno.wtf/api')
                        .then(function (response) {
                            console.log(response);
                            //capitalize 把首个字母开头变成大写，其余变成小写
                            vm.answer = _.capitalize(response.data.answer)
                        })
                        .catch(function (error) {
                            vm.answer = 'Error! Could not reach the API. ' + error
                        })
                }
            }
        })
    </script>
</body>

</html>
```

#### watch与computed什么时候用？

- 当有异步操作时，computed没法操作，得用watch。
- 只是计算一个值的结果，就用computed。