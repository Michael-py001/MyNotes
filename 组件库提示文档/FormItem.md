## FormItem 组件使用说明文档

> \-  `FormItem` 组件是基于 `ant-design-vue` 的 `form-item` 组件进行封装的，支持 `ant-design-vue` 的大部分表单组件，同时也支持自定义组件。

- `FormItem` 组件单独使用时，适用于少量表单控件的页面。 对于表单控件多的页面，更推荐使用 `FormPage` 组件开发。

- `FormPage` 组件基于 `FormItem` 组件进行封装，可以通过 JS 数组配置大量复杂的表单配置，更加方便构建大型表单页面。

## **基础用法**

 使用 `FormItem` 组件需要在外层用 `Form` 组件或者官方 `<a-form>` 组件包裹。

可以通过 `Form` 组件传入 `model` 和 `rules` 属性，`model` 为表单数据对象，`rules` 为表单验证规则。

```vue
<template>
    <Form ref="formRef" :model="formData" :rules="rules">
        <FormItem label="项目主办方" type="text" name="projectHost"></FormItem>
        <FormItem label="项目名称" type="input" name="projectName"></FormItem>
        <FormItem
            label="项目区域"
            type="a-select"
            name="projectzone"
            :options="[
                { label: '项目区域A', value: 'A' },
                { label: '项目区域B', value: 'B' },
            ]"
        ></FormItem>
        <FormItem label="项目时间" type="date-picker" name="projectTime"></FormItem>
        <FormItem label="项目生效" type="switch" name="available" :form-data="formData"></FormItem>
        <FormItem
            label="项目类型"
            type="checkbox-group"
            name="projectType"
            :options="[
                { label: '项目类型A', value: 'A' },
                { label: '项目类型B', value: 'B' },
            ]"
        ></FormItem>
        <FormItem
            label="项目期限"
            type="radio-group"
            name="projectTimeLimit"
            :options="[
                { label: '5天', value: '5' },
                { label: '10天', value: '10' },
            ]"
        ></FormItem>
        <FormItem label="项目描述" type="textarea" name="projectDesc"></FormItem>
    </Form>
    <div class="btns">
        <a-space :size="30">
            <a-button type="primary" @click="sumbit">提交</a-button>
            <a-button>取消</a-button>
        </a-space>
    </div>
</template>

<script setup>
import { ref } from 'vue'
import { FormItem, Form } from 'x-widgets'

const formRef = ref(null)

const formData = ref({
    projectHost: '主办方',
    projectName: null,
    projectzone: null,
    projectTime: null,
    available: true,
    projectType: [],
    projectTimeLimit: null,
    projectDesc: null,
})

const rules = {
    projectName: [{ required: true, message: '请输入项目名称' }],
    projectzone: [{ required: true, message: '请选择项目区域' }],
    projectTime: [{ required: true, message: '请选择项目时间' }],
    projectType: [{ required: true, message: '请选择项目类型' }],
    projectTimeLimit: [{ required: true, message: '请选择项目期限' }],
    projectDesc: [{ required: true, message: '请输入项目描述' }],
}

function sumbit(val) {
    formRef.value.validate().then(res => {
        alert('提交成功')
    })
}
</script>

<style scoped>
.btns {
    text-align: center;
}
</style>
```

## **其他写法**

除了上面的写法，还可以直接在 `FormItem` 组件中传入 `rules` 和 `formData` 属性。

```vue
<template>
    <Form ref="formRef">
        <FormItem
            label="项目名称"
            type="input"
            name="projectName"
            :formData="formData"
            :rules="[{ required: true, message: '请输入项目名称' }]"
        ></FormItem>
        <FormItem
            label="项目区域"
            type="a-select"
            name="projectzone"
            :options="[
                { label: '项目区域A', value: 'A' },
                { label: '项目区域B', value: 'B' },
            ]"
            :formData="formData"
            :rules="[{ required: true, message: '请选择项目区域' }]"
        ></FormItem>
    </Form>
</template>
```

## **FormItem Props**

| 参数 | 说明 | 类型 | 默认值 |
| --- | --- | --- | --- |
| type | 组件的类型，见下方[组件类型](#formitem-组件支持的类型) | _string_ |  |
| formData | 表单数据对象，必须是响应式数据 | _object_ |  |
| attributes | 给外层`ant-design-vue`的`form-item`组件设置的属性，具体属性值可参考[Form.Item 参数](https://2x.antdv.com/components/form-cn#API) | _object_ |  |
| props | 给当前`type`类型组件设置的html标签属性，如 `style` ， `disabled` 等 | _object_ |  |
| labelAlign | label 标签的文本对齐方式 | _'left' \| 'right'_ | `'left'` |
| align | `form-item`的对齐方式 | _'left' \| 'center' \| 'around'_ | `'left'` |
| style | `form-item`外层 style 样式属性 | _object_ |  |
| name | 表单名称 | _string_ |  |
| beforeText | 给`form-item`前面添加的文字 | _string_ |  |
| afterText | 给`form-item`后面添加的文字 | _string_ |  |
| options | options 数据，适用于`select`,`radio-group`,`check-box-group`等类型组件。 | _array<{label,value}>_ | `[]` |
| valueKey | 用于修改 options 数据中 key 值 | _string_ | `value` |
| labelKey | 用于修改 options 数据中 label 值 | _string_ | `label` |
| label | label 标签的文本 | _string \| slot_ |  |
| event | 给当前`type`类型组件设置的事件 | _object_ |  |
| rules | 表单验证规则 | _object \| array_ |  |
| textBold | 给`text`类型组件设置文本加粗 | _boolean_ | `false` |
| disabled | 禁用当前组件 | _boolean_ | `false` |
| autoLabelHeight | 当`label`显示空间不足时，可设置为`true`，`label`会自适应高度显示 | _boolean_ | `false` |
| hideRequiredMark | 隐藏当前字段的`*`必选符号 | _boolean_ | `false` |
| addRequiredMark | 给当前字段添加`*`必选符号 | _boolean_ | `false` |
| formSlot | 自定义表单组件，适用于自定义组件 | _string_                         |  |
| afterSlot | 自定义表单组件后面的内容，适用于自定义组件 | _string_                         |  |
| beforeSlot | 自定义表单组件前面的内容，适用于自定义组件 | _string_                         |  |

以下是props的代码内容:

```ts
import type { ExtractPropTypes, PropType, Ref } from 'vue'
import { InputProps, SelectProps, CheckboxProps, RadioProps } from 'ant-design-vue'
export type formItemTypes =
    | 'text'
    | 'select'
    | 'textarea'
    | 'input-number'
    | 'input'
    | 'time-picker'
    | 'date-picker'
    | 'time-range-picker'
    | 'checkbox-group'
    | 'radio-group'
    | 'city-picker'
export const formItemProps = () => {
    return {
        name: {
            type: [String, Number, Array] as PropType<string | number | Array<string | number>>,
            default: '',
        },
        rules: {
            type: Array as PropType<Array<Record<string, any>>>,
            default: null,
        },
        label: {
            type: String as PropType<string>,
            default: '',
        },
        labelAlign: {
            type: String as PropType<'left' | 'center' | 'right'>,
            default: 'right',
        },
        formData: {
            type: Object as PropType<Ref<Record<string, any>> | null>,
            default: null,
        },
        attributes: {
            type: Object as PropType<Record<string, any>>,
            default: null,
        },
        props: {
            type: Object as PropType<Record<string, any>>,
            default: () => ({}),
        },
        validateInfo: {
            type: Object as PropType<Record<string, any>>,
            default: () => ({}),
        },
        cityValidateInfo: {
            type: Object as PropType<Record<string, any>>,
            default: () => ({}),
        },
        style: {
            type: Object as PropType<Record<string, any>>,
            default: () => ({}),
        },
        //同type , 兼容老版本
        el: {
            type: String as PropType<formItemTypes>,
            default: '',
        },
        type: {
            type: String as PropType<formItemTypes>,
            default: '',
        },
        cityPickerData: {
            type: Object as PropType<Record<string, any>>,
            default: () => ({}),
        },
        beforeText: {
            type: String as PropType<string>,
            default: '',
        },
        afterText: {
            type: String as PropType<string>,
            default: '',
        },
        options: {
            type: Array as PropType<Array<Record<string, any>>>,
            default: () => [],
        },
        valueKey: {
            type: String as PropType<string>,
            default: 'value',
        },
        labelKey: {
            type: String as PropType<string>,
            default: 'label',
        },
        align: {
            type: String as PropType<'left' | 'center' | 'around'>,
            default: 'left',
        },
        event: {
            type: Object as PropType<Record<string, any>>,
            default: () => ({}),
        },
        textBold: {
            type: Boolean as PropType<boolean>,
            default: false,
        },
        autoLabelHeight: {
            type: Boolean as PropType<boolean>,
            default: false,
        },
        disabled: {
            type: Boolean as PropType<boolean>,
            default: false,
        },
        hideRequiredMark: {
            type: Boolean as PropType<boolean>,
            default: false,
        },
        addRequiredMark: {
            type: Boolean as PropType<boolean>,
            default: false,
        },
        textValue: {
            type: [String, Number, Boolean] as PropType<string | number | boolean>,
            default: '',
        },
        unitStyle: {
            type: [String, Object] as PropType<string | Record<string, any>>,
            default: () => ({}),
        },

        // 支持Antdv - FormItem本身的全部属性
        selfProps: {
            type: Object as PropType<Record<string, any>>,
            // type: Object as PropType<Record<string, any>>,
            default: () => ({}),
        },

        // 支持FormItem插槽内Antdv表单组件的全部属性
        slotProps: {
            type: Object as PropType<InputProps & SelectProps & CheckboxProps & RadioProps & any>,
            default: (): any => ({}),
        },
        afterSlot: {
            type: String as PropType<string>,
            default: '',
        },
        beforeSlot: {
            type: String as PropType<string>,
            default: '',
        },
        formSlot: {
            type: String as PropType<string>,
            default: '',
        },
    }
}

export type CustomFormItemProps = Partial<ExtractPropTypes<ReturnType<typeof formItemProps>>>

export interface ColumnForFormItemType extends CustomFormItemProps {
    // city-picker的数据
    columnGroup?: Array<Record<string, any>>
    //插槽名称
    slot?: string
    colspan?: number
}

```



## **FormItem 组件支持的类型**

目前 `type` 支持的类型有：

| type 类型 | 说明 |
| --- | --- |
| text | 自定义 纯文本组件 |
| city-picker | 自定义 地区联动选择器组件 ，最多支持省、市、区、街道 |
| select | 基于官方 [select 选择器](https://2x.antdv.com/components/select-cn) |
| textarea | 基于官方 [Input 文本域](https://2x.antdv.com/components/input-cn#components-input-demo-textarea) |
| input-number | 基于官方 [InputNumber 数字输入框](https://2x.antdv.com/components/input-number-cn) |
| input | 基于官方 [Input 输入框](https://2x.antdv.com/components/input-cn) |
| time-picker | 基于官方 [TimePicker 时间选择框](https://2x.antdv.com/components/time-picker-cn) |
| date-picker | 基于官方 [DatePicker 日期选择框](https://2x.antdv.com/components/date-picker-cn) |
| range-picker | 基于官方 [DatePicker 日期选择框](https://2x.antdv.com/components/date-picker-cn)中的`range-picker`日期范围选择器 |
| checkbox | 基于官方 [Checkbox 多选框](https://2x.antdv.com/components/checkbox-cn)中的`checkbox-group` |
| radio-group | 基于官方 [Radio 单选框](https://2x.antdv.com/components/radio-cn)中的`radio-group` |

