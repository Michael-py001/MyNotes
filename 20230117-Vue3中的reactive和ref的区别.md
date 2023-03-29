# 20230117-Vue3中的reactive和ref的区别

## reactive()

> 只能给对象类型创建响应式对象

返回一个对象的响应式代理。

返回的对象以及其中嵌套的对象都会通过 [ES Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) 包裹，因此**不等于**源对象，**建议只使用响应式代理**，避免使用原始对象。

`reactive()` 返回的是一个原始对象的 [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)，它和原始对象是不相等的：

```js
const raw = {}
const proxy = reactive(raw)

// 代理对象和原始对象不是全等的
console.log(proxy === raw) // false
```

只有代理对象是响应式的，更改原始对象不会触发更新。因此，使用 Vue 的响应式系统的最佳实践是 **仅使用你声明对象的代理版本**。

依靠深层响应性，响应式对象内的嵌套对象依然是代理:

```js
const proxy = reactive({})

const raw = {}
proxy.nested = raw

console.log(proxy.nested === raw) // false
```

## `reactive()` 的局限性

`reactive()` API 有两条限制：

1. 仅对对象类型有效（对象、数组和 `Map`、`Set` 这样的[集合类型](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects#使用键的集合对象)），而对 `string`、`number` 和 `boolean` 这样的 [原始类型](https://developer.mozilla.org/zh-CN/docs/Glossary/Primitive) 无效。

2. 因为 Vue 的响应式系统是通过属性访问进行追踪的，因此我们必须始终保持对该响应式对象的相同引用。这意味着我们不可以随意地“替换”一个响应式对象，因为这将导致对初始引用的响应性连接丢失：

   ```js
   let state = reactive({ count: 0 })
   
   // 上面的引用 ({ count: 0 }) 将不再被追踪（响应性连接已丢失！）
   state = reactive({ count: 1 })
   ```

同时这也意味着当我们将响应式对象的属性，**赋值**或**解构**至本地变量时，或是将该属性**传入一个函数**时，我们会失去响应性：

```js
const state = reactive({ count: 0 })

// n 是一个局部变量，同 state.count
// 给变量赋值，失去响应性连接
let n = state.count
// 不影响原始的 state, state的count值不会变化
n++
// 输出 n: 1
// state1: Proxy {count: 0}

// count 也和 state.count 失去了响应性连接
let { count } = state
// 不会影响原始的 state
count++

// 该函数接收一个普通数字，并且
// 将无法跟踪 state.count 的变化
callSomeFunction(state.count)
```

## ref()

> 可以理解为可以创建任何数据类型的响应式对象，取值要`.value`，reactive只能创建对象类型的响应式对象。

接受一个内部值，返回一个响应式的、可更改的 ref 对象，此对象只有一个指向其内部值的属性 `.value`。

`reactive()` 的种种限制归根结底是因为 JavaScript 没有可以作用于所有值类型的 “引用” 机制。为此，Vue 提供了一个 [`ref()`](https://cn.vuejs.org/api/reactivity-core.html#ref) 方法来允许我们创建可以使用任何值类型的响应式 **ref**：

```js
import { ref } from 'vue'

const count = ref(0)
```

`ref()` 将传入参数的值包装为一个带 `.value` 属性的 ref 对象：

```js
const count = ref(0)

console.log(count) // { value: 0 }
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

和响应式对象的属性类似，ref 的 `.value` 属性也是响应式的。同时，当值为对象类型时，会用 `reactive()` 自动转换它的 `.value`。

一个包含对象类型值的 ref 可以响应式地替换整个对象：

```js
const objectRef = ref({ count: 0 })

// 这是响应式的替换
objectRef.value = { count: 1 }
```

ref 被传递给函数或是从一般对象上被解构时，不会丢失响应性：

```js
const obj = {
  foo: ref(1),
  bar: ref(2)
}

// 该函数接收一个 ref
// 需要通过 .value 取值
// 但它会保持响应性
callSomeFunction(obj.foo)

// 仍然是响应式的
const { foo, bar } = obj
```

简言之，`ref()` 让我们能创造一种对任意值的 “引用”，并能够在不丢失响应性的前提下传递这些引用。这个功能很重要，因为它经常用于将逻辑提取到 [组合函数](https://cn.vuejs.org/guide/reusability/composables.html) 中。

## 实际应用的使用区别

使用`reactive`或者`ref`创建的响应式对象，都能在template模板中使用，且保持响应式。

- ### 区别一：

  reactive创建的对象，比如obj，需要对这个obj里面的属性进行修改，比如obj.a = 1。

​		而使用ref创建的值，比如obj，需要obj.value进行修改。

- ### 区别二：

  reactive只能创建对象，ref可以创建所有类型，包括原始值类型，对象



```js
//使用reactive创建
let DataState = reactive({
  data:[]
})

//修改值时
DataState.data = 'xxx'

//另外一种情况
let DataState2 = reactive({})
//如果此时直接赋值，会失去响应式
DataState2 = '{a:'xxx'}'
```

```js
//使用ref创建
let data = ref([])

//修改值时
data.value = 'xxx'
```

模板上用的区别：

```html
<div>reactive: {{ DataState.data }}</div>
<div>ref: {{ data }}</div>
```

