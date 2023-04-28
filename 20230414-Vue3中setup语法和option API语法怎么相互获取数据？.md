# 20230414-Vue3中setup语法和option API语法怎么相互获取数据？

在组件封装中，需要知道这个组件是否被作为嵌套插槽使用

```html
<FormPage>
  <template #footer>
   <FormPage>xxx</FormPage>
 </template>
</FormPage>
```

1、因为setup语法获取不了$parent，故使用普通script语法获取$parent

```vue
<script>
export default {
  name: 'FormPage',
  data() {
    return {
      isSlot: false, //是否插槽
    };
  },
  created() {
    //具体有多少层$parent需要自己测试，因为组件中可能使用了其他ui组件包裹
    if (this.$parent?.$parent?._name === 'formPage') {
      this.isSlot = true;
    }
  },
};
</script>
```

2、在setup语法中使用`getCurrentInstance`获取组件实例，`currentInstance.ctx`可以获取data或者props属性，从而拿到了普通script上的isSlot属性

![image-20230414154438887](https://s2.loli.net/2023/04/14/dfelgiqBCc8KwAR.png)

```js
<sctipt setup>
    import { getCurrentInstance } from 'vue';
const currentInstance = getCurrentInstance();
    
    // 是否作为插槽组件 兼容props.asChild优先
const isChild = currentInstance.ctx.isSlot;
    
	defineExpose({
      _name: 'FormPage',
    });
</script>
```

