## SearchForm 组件使用说明

组件概述：SearchForm 是一个搜索表单组件。

通过 传入`:formData` 绑定搜索表单的数据，通过 `column` 定义搜索表单的表单项。

## 组件源码

```tsx
/**
 * @description: 搜索表单组件
   @name: SearchForm
 * @author WuShiLi
 */

import FormItem from '../FormItem/FormItem'
import { defineComponent, ref, reactive, watch, onMounted, unref, computed, nextTick } from 'vue'
import { Form, Button, Row, Col } from 'ant-design-vue'
import { CaretUpOutlined, CaretDownOutlined } from '@ant-design/icons-vue'
import type { SearchFormProps, RulesType, FormDataType } from './types/props'
import { searchFormProps } from './types/props'
import { useFormTools } from '../../utils/useFormTools'
import './styles/SearchForm.less'
// import svg from './images/down.svg'
export const PARAMS_KEY = 'ANTDV_TABLE_PARAMS'

export default defineComponent<SearchFormProps>({
    name: 'XSearchForm',
    props: {
        ...searchFormProps(),
    },
    emits: ['search', 'reset', 'update:formData', 'expand'],
    setup(props, { attrs, slots, expose, emit }) {
        const useForm = Form.useForm
        const formRef = ref()
        const wrapperRef = ref()
        const compFormData: FormDataType = computed(() => props.formData)
        const {
            createSearchFormRules,
            checkDisplay,
            cityPickerData,
            cityPickerNames,
            getSelectedAddressName,
        } = useFormTools()
        // 搜索
        const handleSearch = () => {
            validate()
                .then(values => {
                    if (sessionStorage.getItem(PARAMS_KEY)) {
                        sessionStorage.removeItem(PARAMS_KEY)
                    }
                    emit('search', values)
                })
                .catch(error => {
                    throw new Error(JSON.stringify(error))
                })
        }

        // 重置
        const handleReset = () => {
            resetFields()
            if (sessionStorage.getItem(PARAMS_KEY)) {
                sessionStorage.removeItem(PARAMS_KEY)
            }
            emit('reset', unref(compFormData))
        }

        // 对外暴露省市区
        let cityPicker = ref([])
        watch(
            () => props.disabled,
            newVal => {
                if (newVal === undefined) return
                props.column.forEach(item => {
                    if (item.disabled === undefined) {
                        item.props = item.props || {}
                        Reflect.set(item.props, 'disabled', newVal)
                    }
                })
            },
            {
                immediate: true,
            },
        )
        const rules: RulesType = reactive(createSearchFormRules(props))

        let { resetFields, validate, validateInfos, clearValidate } = useForm(props.formData, rules)

        expose({
            handleReset,
            handleSearch,
            form: formRef,
            formRef,
            validate,
            clearValidate,
            resetFields,
            getSelectedAddressName: name => getSelectedAddressName(name, cityPicker.value),
        })

        // 展开收起
        const hasMore = ref(false)
        const showMore = ref(false) // 是否展开
        let debounce = false
        let timer = null

        function handleShowMore() {
            showMore.value = !showMore.value
            nextTick(() => {
                emit('expand', showMore.value)
            })
        }
        const rowRef = ref<any>()
        let innerHeight = 0
        let resizeHandle = () => {}
        onMounted(() => {
            const wrapper = wrapperRef.value
            const antForm = wrapper.querySelector('.ant-form')
            let formHeight = antForm.offsetHeight
            resizeHandle = () => {
                if (debounce) return
                debounce = true
                if (timer) {
                    clearTimeout(timer)
                }

                timer = setTimeout(() => {
                    if (wrapper) {
                        let formItem = antForm.firstChild
                        if (formItem && formItem.nodeType !== 1) {
                            formItem = formItem.nextSibling
                        }
                        let formOverHeight = formItem.offsetHeight

                        if (formOverHeight / formHeight >= 2) {
                            hasMore.value = true
                        } else {
                            hasMore.value = false
                        }
                    }
                    debounce = false
                }, 300)
            }
            props.enabledExpand && resizeHandle() //开启展开收起功能
            window.addEventListener('resize', resizeHandle)
            innerHeight = rowRef.value?.$el?.offsetHeight
        })

        watch(
            compFormData,
            () => {
                if (rowRef.value?.$el?.offsetHeight !== innerHeight) {
                    resizeHandle()
                }
            },
            {
                deep: true,
            },
        )

        return () => {
            return (
                <div
                    class={`x-search-form ${
                        props.enabledExpand && hasMore.value ? 'showMore' : ''
                    }`}
                    ref={wrapperRef}
                >
                    {props.enabledExpand && hasMore.value ? (
                        <span class="x-search-form-more-icon" onClick={handleShowMore}>
                            {showMore.value ? <CaretUpOutlined /> : <CaretDownOutlined />}
                        </span>
                    ) : null}
                    <Form
                        ref={formRef}
                        class={`x-search-form-field ${
                            showMore.value || !props.enabledExpand ? 'showMore' : ''
                        }`}
                        model={compFormData.value}
                        layout={props.layout}
                        {...attrs}
                    >
                        <Row gutter={props.gutter} ref={rowRef}>
                            {props.column.map((item, index) => {
                                const { props: _props } = item as any
                                if (item.slot) {
                                    return <Col span={item.colspan}>{slots[item.slot]()}</Col>
                                } else if (item.el || item.type) {
                                    const Item = (
                                        <FormItem
                                            key={index}
                                            ref={el => {
                                                if (cityPickerNames.includes(item.el)) {
                                                    if (!cityPicker.value.includes(el)) {
                                                        cityPicker.value.push(el)
                                                    }
                                                }
                                            }}
                                            hide-required-mark={item.hideRequiredMark}
                                            style={item.style || { 'min-width': '250px' }}
                                            label={item.label}
                                            name={item.name}
                                            type={item.el}
                                            props={item.props}
                                            attributes={item.attributes}
                                            options={item.options}
                                            form-data={compFormData.value}
                                            label-align={item.labelAlign || 'right'}
                                            textValue={item.textValue}
                                            align={item.align}
                                            event={item.event}
                                            after-text={item.afterText}
                                            before-text={item.beforeText}
                                            validate-info={validateInfos[item.name as any]}
                                            city-validate-info={validateInfos}
                                            city-picker-data={cityPickerData.value}
                                            disabled={_props?.disabled || item.disabled}
                                            selfProps={item.selfProps}
                                            slotProps={item.slotProps}
                                            afterSlot={item.afterSlot}
                                            beforeSlot={item.beforeSlot}
                                            formSlot={item.formSlot}
                                            unitStyle={item.unitStyle}
                                        >
                                            {{
                                                ...slots,
                                            }}
                                        </FormItem>
                                    )
                                    if (checkDisplay(item)) {
                                        return <Col span={item.colspan}>{Item}</Col>
                                    }
                                    return null
                                }
                            })}
                        </Row>
                    </Form>
                    {props.showButtons && (
                        <div class="x-search-form-btns">
                            {props.resetText && (
                                <Button
                                    type="default"
                                    disabled={props.disabled || props.loading}
                                    loading={props.loading}
                                    onClick={handleReset}
                                    class="x-search-form-btn-reset"
                                >
                                    {props.resetText}
                                </Button>
                            )}
                            {props.searchText && (
                                <Button
                                    type="primary"
                                    disabled={props.disabled || props.loading}
                                    loading={props.loading}
                                    onClick={handleSearch}
                                    class="x-search-form-btn-search"
                                >
                                    {props.searchText}
                                </Button>
                            )}
                        </div>
                    )}
                    {slots?.default?.()}
                </div>
            )
        }
    },
})

```

## 基本用法

```vue
<template>
    <SearchForm
        v-model:formData="formData"
        :column="column"
        @reset="GetList"
        @search="GetList"
    ></SearchForm>
</template>

<script setup>
import { SearchForm } from 'x-widgets'
import useSearchForm from './composables/useSearchForm.js'

const { column, formData } = useSearchForm()

function GetList(params) {
    alert(JSON.stringify(params))
}
</script>
```

```js
// useSearchForm.js
import { reactive, ref } from 'vue'
export default function useSearchForm() {
    const formData = reactive({
        name: null,
        phone: null,
        sex: null,
    })
    const column = ref([
        {
            el: 'input',
            label: '姓名',
            name: 'name',
        },
        {
            el: 'input',
            label: '电话',
            name: 'phone',
        },
        {
            el: 'select',
            label: '性别',
            name: 'sex',
            options: [
                {
                    label: '男',
                    value: '男',
                },
                {
                    label: '女',
                    value: '女',
                },
            ],
        },
    ])

    return {
        column,
        formData,
    }
}
```

## **与表格组件配合使用**

条件筛选通常会与表格同时出现

```vue
<template>
    <div>
        <SearchForm
            v-model:formData="formData"
            :column="column"
            @reset="GetList"
            @search="GetList"
        ></SearchForm>
        <Table
            :columns="columns"
            :dataSource="list"
            :pagination="pagination"
            :loading="loading"
            :scroll="{ x: 600 }"
        >
            <template #operation="{ record }">
                <a-space>
                    <a-button type="link" @click="confirm(record)">选择</a-button>
                </a-space>
            </template>
        </Table>
    </div>
</template>

<script setup>
import Table from '../Table.vue'
import SearchForm from '../../SearchForm/SearchForm.vue'
import useTable from './composables/useTable.js'
import useSearchForm from './composables/useSearchForm.js'

const { column, formData } = useSearchForm()
const { columns, list, pagination, GetList, RefreshList, loading, confirm } = useTable(formData)
</script>

```

```js
// useTable.js

import useGetList from '../../../../hooks/useGetList'
import { ref, onMounted } from 'vue'
import { getUserList } from './api'
export default function useTable(params) {
    const columns = ref([
        {
            title: '姓名',
            dataIndex: 'name',
            width: 120,
            align: 'center',
        },
        {
            title: '电话',
            dataIndex: 'phone',
            width: 150,
            align: 'center',
        },
        {
            title: '年龄',
            dataIndex: 'age',
            width: 80,
            align: 'center',
        },
        {
            title: '性别',
            dataIndex: 'sex',
            width: 80,
            align: 'center',
        },
        {
            title: '地址',
            dataIndex: 'address',
            width: 150,
            align: 'center',
        },
        {
            title: '操作',
            width: 120,
            align: 'center',
            fixed: 'right',
            slots: {
                customRender: 'operation',
            },
        },
    ])

    // 添加空值时显示
    columns.value
        .filter(i => i.title !== '操作')
        .forEach(item => {
            item['customRender'] = ({ text, record, index }) => {
                let result = '--'
                if (text) {
                    result = text
                }
                return {
                    children: result, //显示的值
                }
            }
        })

    function confirm(record) {
        alert(`你选择了${record.name}`)
    }

    const { GetList, list, pagination, loading, currentParams, RefreshList } = useGetList(
        getUserList,
        params,
    )

    onMounted(() => {
        GetList()
    })

    return {
        columns,
        pagination,
        list,
        GetList,
        loading,
        RefreshList,
        currentParams,
        confirm,
    }
}

```

```js
// useSearchForm.js

import { reactive, ref } from 'vue'
export default function useSearchForm() {
    const formData = reactive({
        name: null,
        phone: null,
        sex: null,
    })
    const column = ref([
        {
            el: 'input',
            label: '姓名',
            name: 'name',
        },
        {
            el: 'input',
            label: '电话',
            name: 'phone',
        },
        {
            el: 'select',
            label: '性别',
            name: 'sex',
            options: [
                {
                    label: '男',
                    value: '男',
                },
                {
                    label: '女',
                    value: '女',
                },
            ],
        },
    ])

    return {
        column,
        formData,
    }
}

```

## **禁用状态**

禁用状态时，表单项不可编辑，搜索和重置按钮不可点击。

```vue
<template>
    <a-switch v-model:checked="disabled" checked-children="禁用中"></a-switch>
    <hr />
    <SearchForm
        v-model:formData="formData"
        :column="column"
        :loading="loading"
        :disabled="disabled"
        @reset="GetList"
        @search="GetList"
    ></SearchForm>
</template>

<script setup>
import { ref } from 'vue'
import SearchForm from '../SearchForm.vue'
import useSearchForm from './composables/useSearchForm.js'

const { column, formData } = useSearchForm()

const loading = ref(false)
const disabled = ref(true)

function GetList(params) {}
</script>

```

