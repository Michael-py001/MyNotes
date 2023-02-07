# 20230207-Vue3中computed不生效的问题

> [vue的computed如果没有出现在模板里面，当它依赖的响应式属性发生变化，getter会触发吗？_天地会珠海分舵的博客-CSDN博客](https://blog.csdn.net/zhubaitian/article/details/127729739)

## 一、监听依赖的值使用ref定义

```js
let activeItem = ref('')

const isActive = computed(() => {
    const result = activeItem.value?.userStatusInt === 1 ? true : false
    return result
});
```

如果`activeItem`使用reactive定义，那么后面给activeItem重新赋值时，已经脱离原本的响应式值了，是重新赋值一个新的内存地址给它，所以computed内不会触发。

```js
let activeItem = reactive('')
activeItem = record // 响应式断开
```

## 二、需要在模板上引用computed的值

上面的`isActive`值必须要在模板中引用，如果不引用则无法收集依赖，页面中没有访问computed的值，也就无法触发其getter。

而且传递给模板使用时，不能传`isActive.value`，而是要传原始的ref值：`isActive`，不然也不会触发computed。