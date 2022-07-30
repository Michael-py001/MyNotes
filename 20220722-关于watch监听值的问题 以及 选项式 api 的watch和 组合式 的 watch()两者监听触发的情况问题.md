# 关于watch监听值的问题 以及 选项式 api 的watch和 组合式 的 watch()两者监听触发的情况问题

## 组件代码

父组件

```html
<template>
  <HelloWorld :arr = "arr"/>
</template>
<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup
import HelloWorld from './components/HelloWorld.vue'
import {ref} from 'vue'
let arr = ref([])

setTimeout(()=>{
  console.log('直接赋值')
  arr.value = [1,2,3]
},1500)
</script>
```

子组件

```html
<script setup>
import { ref ,watch } from 'vue'

const props = defineProps({
  arr: Array
})
console.log("props:",props)
//
watch(props.arr,(arr)=>{
  console.log("直接取值,arr改变：",arr)
})
watch(()=>props.arr,(arr)=>{
  console.log("函数返回值,arr改变：",arr)
})
</script>

<script>
  export default {
    props:{
      arr:Array
    },
    watch: {
      arr(value) {
        console.log("option选项的watch：",value)
      }
    }
  }
</script>

<template>
  arr --> {{arr}}
</template>

<style scoped>
a {
  color: #42b983;
}
</style>

```

## 直接赋值新数组给arr，如何使用watch监听会触发

父组件

```js
setTimeout(()=>{
  console.log('HelloWorld-直接赋值')
  arr.value = [1,2,3]
},1500)
```

子组件

```js
watch(props.arr,(arr)=>{
  console.log("HelloWorld-->使用props.arr：",arr) //不触发
})
watch(()=>props.arr,(arr)=>{
  console.log("HelloWorld-->使用函数返回值,arr改变：",arr)//触发
})

<script>
  export default {
    props:{
      arr:Array
    },
    watch: {
      arr(value) {
        console.log("option选项的watch：",value) //触发
      }
    }
  }
</script>
```

- 这种情况监听`()=>props.arr`触发;

- 监听`props.arr`不触发

- 选项式watch触发

![image-20220722161056317](https://s2.loli.net/2022/07/22/lXaDOV53PsUJIcq.png)

## arr使用push()，如何使用watch监听会触发

```js
setTimeout(()=>{
  console.log("-----------------------")
   console.log('HelloWorldTwo-使用push')
  arr2.value.push(1)
},2500)
```

- 这种情况监听`()=>props.arr`不会触发;

- 监听`props.arr`触发

- 选项式watch不触发

![image-20220722161118282](https://s2.loli.net/2022/07/22/r5quDRjVn2ZcMxQ.png)