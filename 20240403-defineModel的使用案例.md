# 20240403-defineModel的使用案例

> 在antd vue 2.x 版本中，密码输入组件的功能太过鸡肋，自己实现一下密码显示切换的功能。
>
> 使用vue3.4+版本新增的`defineModel`封装一个`InputPassword`组件。

## 子组件

```vue
<template>
    <a-input v-model:value="innerValue" v-bind="$attrs" :type="visible?'text':'password'">
        <template #suffix>
            <EyeOutlined style="color: rgba(0, 0, 0, 0.45)" @click="handleClick" v-if="visible"/>
            <EyeInvisibleOutlined style="color: rgba(0, 0, 0, 0.45)" @click="handleClick" v-else="visible"/>
        </template>
    </a-input>
</template>

<script setup>
import { EyeOutlined, EyeInvisibleOutlined } from '@ant-design/icons-vue'

const emit = defineEmits(['update:value','update:visible'])

const props = defineProps({
    value: {
        type: String,
        default: '',
    },
})

const innerValue = computed({
    get: () => props.value,
    set: val => {
        emit('update:value', val)
    },
})

const visible = defineModel('visible', {
    type: Boolean,
    default: false,
})

function handleClick() {
    visible.value = !visible.value
}
</script>

```

### 实现双向绑定

```js
const innerValue = computed({
    get: () => props.value,
    set: val => {
        emit('update:value', val)
    },
})
```

使用computed的set方法更新父组件的`v-model:value`值。也可以使用`defineModel`实现。

### 实现密码隐藏显示切换

```vue
 <a-input v-model:value="innerValue" v-bind="$attrs" :type="visible?'text':'password'">
        <template #suffix>
            <EyeOutlined style="color: rgba(0, 0, 0, 0.45)" @click="handleClick" v-if="visible"/>
            <EyeInvisibleOutlined style="color: rgba(0, 0, 0, 0.45)" @click="handleClick" v-else="visible"/>
        </template>
    </a-input>
```

引用两个icon自己做点击切换，input的type跟随visiable切换类型。

```js
const visible = defineModel('visible', {
    type: Boolean,
    default: false,
})
```

visible使用`defineModel`实现，第一个参数是父组件使用时：`v-model:xxx`的`xxx`值；

第二个参数传入一个对象，type指定类型，default指定默认值。

![image-20240403150332709](https://s2.loli.net/2024/04/03/gh8lMAktxF165IL.png)

## 父组件使用

```vue
<InputPassword
               size="large"
               placeholder=""
               :maxlength="16"
               autocomplete="off"
               style="height: 44px"
               v-model:value.trim="pwd_state.loginForm.pwd"
               v-model:visible="pwdVisible"
               />
```

其他属性可以通过`$attrs`透传到子组件。