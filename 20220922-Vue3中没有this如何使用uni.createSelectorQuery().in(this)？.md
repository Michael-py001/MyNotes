# 20220922-Vue3中没有this如何使用uni.createSelectorQuery().in(this)？

​		众所周知，Vue3中使用setup语法的话是没有this的，那么如果要把vue2项目的代码重构成Vue3语法，老项目中的`uni.createSelectorQuery().in(this)`该如何改造？

​		答案是使用`getCurrentInstance()`，该方法可以获取到当前组件的实例，相当于Vue2中的this。

```js
import {getCurrentInstance} from 'vue'
const instance = getCurrentInstance()
```

