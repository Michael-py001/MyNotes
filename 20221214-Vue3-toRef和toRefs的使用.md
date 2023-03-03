# Vue3-toRef和toRefs的使用

## toRef()

基于响应式对象上的**一个属性**，创建一个对应的 ref。这样创建的 ref 与其源属性保持同步：改变源属性的值将更新 ref 的值，反之亦然。

toRef和toRefs的区别就是，前者是给一个响应式对象中的一个属性，创建对应的ref。后者是给整个响应式对象所有对象都创建ref。

```js
const state = reactive({
  foo: 1,
  bar: 2
})

const fooRef = toRef(state, 'foo')

// 更改该 ref 会更新源属性
fooRef.value++
console.log(state.foo) // 2

// 更改源属性也会更新该 ref
state.foo++
console.log(fooRef.value) // 3
```

请注意，这不同于：

```js
const fooRef = ref(state.foo)
```

上面这个 ref **不会**和 `state.foo` 保持同步，因为这个 `ref()` 接收到的是一个纯数值。

`toRef()` 这个函数在你想把一个 prop 的 ref 传递给一个组合式函数时会很有用：

```html
<script setup>
import { toRef } from 'vue'

const props = defineProps(/* ... */)

// 将 `props.foo` 转换为 ref，然后传入
// 一个组合式函数
useSomeFeature(toRef(props, 'foo'))
</script>
```

当 `toRef` 与组件 props 结合使用时，关于禁止对 props 做出更改的限制依然有效。尝试将新的值传递给 ref 等效于尝试直接更改 props，这是不允许的。在这种场景下，你可能可以考虑使用带有 `get` 和 `set` 的 [`computed`](https://cn.vuejs.org/api/reactivity-core.html#computed) 替代。

## toRefs()

将**一个响应式对象**转换为一个普通对象，这个普通对象的每个属性都是指向源对象相应属性的 ref。每个单独的 ref 都是使用 [`toRef()`](https://cn.vuejs.org/api/reactivity-utilities.html#toref) 创建的。

在转换时，需要用`reactive`创建响应式对象才行。

- **示例**

  js

  ```js
  const state = reactive({
    foo: 1,
    bar: 2
  })
  
  const stateAsRefs = toRefs(state)
  /*
  stateAsRefs 的类型：{
    foo: Ref<number>,
    bar: Ref<number>
  }
  */
  
  // 这个 ref 和源属性已经“链接上了”
  state.foo++
  console.log(stateAsRefs.foo.value) // 2
  
  stateAsRefs.foo.value++
  console.log(state.foo) // 3
  ```

  当从组合式函数中返回响应式对象时，`toRefs` 相当有用。使用它，消费者组件可以解构/展开返回的对象而不会失去响应性：

  ```js
  function useFeatureX() {
    const state = reactive({
      foo: 1,
      bar: 2
    })
  
    // ...基于状态的操作逻辑
  
    // 在返回时都转为 ref
    return toRefs(state)
  }
  
  // 可以解构而不会失去响应性
  const { foo, bar } = useFeatureX()
  ```
  
  `toRefs` 在调用时只会为源对象上可以枚举的属性创建 ref。如果要为可能还不存在的属性创建 ref，请改用 [`toRef`](https://cn.vuejs.org/api/reactivity-utilities.html#toref)。

## toRefs实际使用案例

1. 通过reactive创建响应式对象
2. `toRefs(data)`转换ref
3. return时，通过...展开符把每一个对象成员都暴露出去: `...dataRefs`

```js
setup() {
   const data = reactive({
      column: [
        {
          el: 'a-input',
          label: '资产名称',
          name: 'assetName',
        },
        {
          el: 'cascader',
          label: '组织机构',
          name: 'sysOrganizationId',
        },
        {
          el: 'a-select',
          label: '资源性质',
          name: 'assetNature',
          options: assetNature,
        },
        {
          el: 'a-select',
          label: '资源类型',
          name: 'assetType',
          options: assetType,
          labelKey: 'itemValue',
          valueKey: 'itemCode',
        },
      ],
      isShowModal: false, // 是否显示资产登记弹窗
    });
    const dataRefs = toRefs(data);
    return {
        ...dataRefs
    }
}

```

通过`toRefs`处理后：转换成普通对象，但是值是通过ref创建的，具有响应式。

![image-20221214173720679](https://f.pz.al/pzal/2022/12/14/62dec812eb88a.png)

![image-20221214173736218](https://f.pz.al/pzal/2022/12/14/aab4f8ecec710.png)



## toRef和toRefs的使用案例

`organizationForm`是一个用`reactive`创建的响应式对象。现在需要把这个对象里的`organizationName`传到其他地方使用，在页面上显示。

```js
 const organizationForm = reactive({
     ...,
     organizationName: ''
 })
```

```js
let {xxx} = useFormModal({
  organizationName:organizationForm.organizationName
})
```

使用组合式函数引入后，再传入到其他函数时，是按值传递的，传入的是当时的值，后面`organizationName`内容改变了，在`useFormModal`里面的值也是不会改变的。此时需要把`organizationName`转为响应式值。

### 使用toRef把某个属性值转为响应式

```js
let organizationName = toRef(organizationForm,'organizationName')

let {xxx} = useFormModal({
  organizationName
})
```

### 使用toRefs方案

把`organizationForm`整个转换，再解构出里面的`organizationName`，此时也是响应式的。

```js
let {organizationName} = toRefs(organizationForm)
let {xxx} = useFormModal({
  organizationName
})
```

## unref()

如果参数是 ref，则返回内部值，否则返回参数本身。这是 `val = isRef(val) ? val.value : val` 计算的一个语法糖。

- **示例**

  ```ts
  function useFoo(x: number | Ref<number>) {
    const unwrapped = unref(x)
    // unwrapped 现在保证为 number 类型
  }
  ```