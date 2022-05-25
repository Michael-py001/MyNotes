# 20220422-provide inject在Vue2 Vue3用法及如何做到数据响应式

## provide / inject

> - **类型**：
>   - **provide**：`Object | () => Object`
>   - **inject**：`Array<string> | { [key: string]: string | Symbol | Object }`

官方文档描述：

​		这对选项需要一起使用，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在其上下游关系成立的时间里始终生效。

- `provide` 选项应该是一个对象或返回一个对象的函数
- `inject` 选项应该是：
  - 一个字符串数组，或
  - 一个对象，对象的 key 是本地的绑定名，value 是：
    - 在可用的注入内容中搜索用的 key (字符串或 Symbol)，或
    - 一个对象，该对象的：
      - `from` property 是在可用的注入内容中搜索用的 key (字符串或 Symbol)
      - `default` property 是降级情况下使用的 value

> 提示：`provide` 和 `inject` 绑定并不是可响应的。这是刻意为之的。然而，如果你传入了一个可监听的对象，那么其对象的 property 还是可响应的。

## 实际案例

### 普通对象

```js
provide:{
      shop_type:this.type
    },
```

这种普通对象写法，this.type无法响应式，也就是在父组件type改变了，在子组件里inject里获取的值不是最新的。

### 如何做到返回响应式

- Vue2

```js
provide(){
      return {
        shop_type:()=>this.type
      }
    },
```

返回一个函数，这样在子组件里获取的type是可以响应式的。子组件里使用时，需要执行这个函数，返回type值

- Vue3的响应式处理
  需要引入使用`computed`，如果直接像vue2那样使用返回函数，就只会返回这个函数的字符串

```js
import { computed } from 'vue'

export default {
  data() {
    return {
      message: 'hello!'
    }
  },
  provide() {
    return {
      // explicitly provide a computed property
      message: computed(() => this.message)
    }
  }
}
```

子组件接收：

```js
inject: ['message']
```

使用时：`message.value`才能获取到真实值

使用watch监测：也需要.value，不然不会监测到

```js
watch: {
    'navActive.value'(value) {
        console.log("navActive:",value);
    }
},
```

#### 使用自动解包

> [Provide / Inject | Vue.js (vuejs.org)](https://vuejs.org/guide/components/provide-inject.html#working-with-reactivity)

![image-20220521113612647](https://s2.loli.net/2022/05/21/2PpuLFhKb6X8stx.png)

意思就是如果你的vue版本是3.3以下的，如果想使用自动解包，不需要再加`.value`，需要设置一个参数：

```js
//main.js
app.config.unwrapInjectedRef = true
```

此时，直接使用inject接收的值就是响应式的值了

```js
watch: {
    navActive(value) {
        console.log("navActive:",value);
    }
},
```

