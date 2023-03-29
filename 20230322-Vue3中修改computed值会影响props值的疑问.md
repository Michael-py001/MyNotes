# 20230322-Vue3中修改computed值会影响props值的疑问



有下面一段代码：

```js
const props = defineProps({
  formData: {
    type: Object,
    default: () => ({}),
  }
)

const innerFormData = computed({
  get:(val) =>{
    return props.formData
  },
  set: (val) => {
    console.log("set---:", val);
    emits('update:formData', val)
  }
})
```

如果我修改innerFormData里的值：

```js
innerFormData.value.id = '123' //修改或者新增属性
```

那么`props.formData`也会同步修改。

但是computed中的setter方法此时不会被触发，这是因为在Vue3中，computed属性的setter方法只有在给整个computed属性赋值时才会触发，而不是在修改computed属性的某个属性时触发。因此，当你修改innerFormData的id属性时，不会触发computed属性的setter方法。



下面两种写法都是一样的，修改`innerFormData`上的属性，相当于在修改`props.formData`的属性，会同步修改

```js
const innerFormData = ref(props.formData) //1
const innerFormData = computed({ //2
  get:(val) =>{
    return props.formData
  }
})
```

