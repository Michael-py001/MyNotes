

## FormPage 组件说明文档

组件描述：该组件用于页面表单构建，通过配置js即可渲染出不同的表单控件



## 组件源码

```vue
<!-- 
    @Description: 表单页面组件
		@Name: FormPage
    @Author: WuShiLi
 -->
<template>
    <div
        ref="FormPageContainerRef"
        class="x-formpage-container"
        :style="$attrs.style || FormPageContainerHeight"
    >
        <div v-if="$slots.header" ref="headerRef" class="x-formpage-container-fixed-header">
            <slot name="header"></slot>
        </div>
        <Spin :spinning="loading">
            <div ref="formRef" style="overflow-y: auto" class="formRef">
                <Form
                    ref="formContainerRef"
                    class="x-formpage-container-form"
                    :layout="layout"
                    :autocomplete="autocomplete"
                    :wrapper-col="wrapperCol"
                    :label-col="labelCol"
                    :label-align="labelAlign"
                    v-bind="$attrs"
                    :model="formData"
                    :rules="rules"
                >
                    <fieldset
                        ref="fieldsetRef"
                        :disabled="disabled"
                        :style="fieldsetStyle"
                        class="form-container"
                        :class="{ 'form-container-grid': fieldsetStyle.gridTemplate }"
                    >
                        <template v-for="(model, j) in formModel" :key="j">
                            <!-- :disabled="model.disabled" -->
                            <template v-if="checkDisplay(model)">
                                <fieldset
                                    ref="fieldsetRef"
                                    :disabled="false"
                                    :style="model.fieldsetStyle"
                                >
                                    <template v-if="model.title">
                                        <slot name="modelTitle" :model="model" :title="model.title">
                                            <template v-if="_hasModelTitleComponent">
                                                <component
                                                    is="defaultModelTitleComponent"
                                                    :title="model.title"
                                                ></component>
                                            </template>
                                            <template v-else>
                                                <div v-if="model.divider === false" class="title">
                                                    {{ model.title }}
                                                </div>
                                                <Divider v-else orientation="left" dashed>
                                                    {{ model.title }}
                                                </Divider>
                                            </template>
                                        </slot>
                                    </template>
                                    <div
                                        :style="model.style"
                                        :class="model.class"
                                        v-if="model.display"
                                    >
                                        <Row :gutter="gutter">
                                            <template
                                                v-for="(item, index) in model.column"
                                                :key="index"
                                            >
                                                <Col
                                                    v-if="checkDisplay(item)"
                                                    v-show="isShow(item)"
                                                    :span="item.colspan || colspan"
                                                >
                                                    <template v-if="item.slot">
                                                        <slot :name="item.slot" :item="item"></slot>
                                                    </template>
                                                    <FormItem
                                                        v-else-if="
                                                            item.el || item.type || item.formSlot
                                                        "
                                                        :ref="
                                                            el => {
                                                                if (
                                                                    cityPickerNames.includes(
                                                                        item.el,
                                                                    )
                                                                ) {
                                                                    if (!cityPicker.includes(el)) {
                                                                        cityPicker.push(el)
                                                                    }
                                                                }
                                                            }
                                                        "
                                                        :hide-required-mark="item.hideRequiredMark"
                                                        :addRequiredMark="item.addRequiredMark"
                                                        :style="{
                                                            paddingBottom: verticalGap + 'px',
                                                            ...item.style,
                                                        }"
                                                        :label="item.label"
                                                        :name="item.name"
                                                        :type="item.el || item.type"
                                                        :props="item.props"
                                                        :attributes="item.attributes"
                                                        :options="item.options"
                                                        :form-data="innerFormData"
                                                        :wrapper-col="item.wrapperCol || wrapperCol"
                                                        :city-validate-info="validateInfos"
                                                        :city-picker-data="cityPickerData"
                                                        :textValue="item.textValue"
                                                        :event="item.event"
                                                        :align="item.align"
                                                        :after-text="item.afterText"
                                                        :before-text="item.beforeText"
                                                        :textBold="item.textBold"
                                                        :trim="item.trim"
                                                        :unitStyle="item.unitStyle"
                                                        :disabled="
                                                            item.disabled || item.props?.disabled
                                                        "
                                                        :labelAlign="item.labelAlign || labelAlign"
                                                        :autoLabelHeight="item.autoLabelHeight"
                                                        :selfProps="item.selfProps"
                                                        :slotProps="item.slotProps"
                                                        :afterSlot="item.afterSlot"
                                                        :beforeSlot="item.beforeSlot"
                                                        :formSlot="item.formSlot"
                                                        :afterFormSlot="item.afterFormSlot"
                                                        :rules="rules[item.name]"
                                                        @clearValidate="handleClearValidate"
                                                        @validateName="validateName"
                                                    >
                                                        <template
                                                            v-for="(name, index) in Object.keys(
                                                                slot,
                                                            )"
                                                            #[name]="scopeData"
                                                            :key="index"
                                                        >
                                                            <slot
                                                                :name="name"
                                                                v-bind="scopeData"
                                                                :formData="formData"
                                                                :item="item"
                                                                :key="index"
                                                            ></slot>
                                                        </template>
                                                    </FormItem>
                                                </Col>
                                            </template>
                                        </Row>
                                    </div>
                                </fieldset>
                            </template>
                        </template>
                    </fieldset>
                </Form>
            </div>
        </Spin>
        <div v-if="$slots.footer" ref="footerRef" class="x-formpage-container-fixed-footer">
            <slot name="footer"></slot>
        </div>
    </div>
</template>

<script setup>
import { Row, Spin, Divider, Col } from 'ant-design-vue'
import {
    nextTick,
    useSlots,
    getCurrentInstance,
    ref,
    defineComponent,
    watch,
    isRef,
    computed,
    reactive,
    onMounted,
    unref,
    provide,
    toRefs,
} from 'vue'
import FormItem from '../FormItem/FormItem'
import Form from '../Form/Form'
import { useFormTools } from '../../utils/useFormTools'

const currentInstance = getCurrentInstance()
let { footer: footerSlot, header: headerSlot } = useSlots()
const {
    createFormPageRules,
    checkDisplay,
    isShow,
    cityPickerData,
    cityPickerNames,
    getSelectedAddressName,
} = useFormTools()
const useForm = Form.useForm
const emit = defineEmits(['update:formData', 'confirm', 'cancel', 'update:show'])
defineOptions({
    name: 'XFormPage',
})
const slot = useSlots()

// FormPage的props属性
const props = defineProps({
    // 表单字段模型
    formModel: {
        type: Array,
        default: () => [],
    },
    // 表单数据
    formData: {
        type: Object,
        default: () => ({}),
    },
    // label样式
    labelCol: {
        type: Object,
        default: () => {
            return { style: { minWidth: '100px' } }
        },
    },
    // 控件样式
    wrapperCol: {
        type: Object,
        default: () => {
            return { span: 17 }
        },
    },
    // 标签文本对齐方式
    labelAlign: {
        type: String,
        default: 'left', //'left' | 'right'
    },
    // 是否显示loading
    loading: {
        type: Boolean,
        default: false,
    },
    // 作为子组件
    asChild: {
        type: Boolean,
        default: false,
    },
    // 字段垂直间隔
    verticalGap: {
        type: [String, Number],
        default: '0',
    },
    // modelTitle插槽组件
    modelTitleComponent: {
        type: Object,
        default: () => {
            return {}
        },
    },
    // 表单容器多余高度
    extraHeight: {
        type: [String, Number],
        default: 0,
    },
    // 表单布局
    layout: {
        type: String,
        default: 'horizontal', //'horizontal' | 'vertical' | 'inline'
    },
    //字段colspan
    colspan: {
        type: Number,
        default: 24,
    },
    //表单间隔
    gutter: {
        type: Array,
        default: () => [12, 0],
    },
    // 是否禁用
    disabled: {
        type: Boolean,
        default: undefined,
    },
    fieldsetStyle: {
        type: [Object, String],
        default: () => {
            return {}
        },
    },
    autocomplete: {
        type: String,
        default: 'on',
    },
})
const formContainerRef = ref(null)
// 是否作为插槽组件 兼容props.asChild优先
let isChild = ref(false)


const innerFormData = computed({
    get: () => props.formData,
})

// 对外暴露省市区
let cityPicker = ref([])

setDisabledWatch()
const rules = reactive(createFormPageRules(props))

function _validate(nameList) {
    return formContainerRef?.value.validate(nameList)
}

function _clearValidate(nameList) {
    return formContainerRef?.value?.clearValidate(nameList)
}

// 创建表单容器
let { validateInfos } = useForm(props.formData, rules)

//校验单个字段
function validateName(name) {
    _validate([name])
}
//清除表单字段校验
function handleClearValidate(name) {
    _clearValidate([name])
}

//恢复表单字段校验
async function resetValidate(name) {
    _clearValidate([name])
}

function setDisabledWatch() {
    watch(
        () => props.disabled,
        (newVal, oldValue) => {
            if (newVal === undefined) return
            props.formModel.forEach(item => {
                item.column.forEach(i => {
                    if (i.props?.disabled !== undefined) {
                        i.props = i.props || {}
                        Reflect.set(i.props, 'disabled', newVal)
                    }
                })
            })
        },
        {
            immediate: true,
        },
    )
}

// 获取footer高度
let footerRef = ref(null),
    headerRef = ref(null),
    formRef = ref(null),
    FormPageContainerRef = ref(null)

let footerHeight = ref(0)
const headerHeight = computed(() => {
    return headerRef.value?.offsetHeight || 0
})
const FormPageContainerHeight = computed(() => {
    let height = 'auto'
    if (props.extraHeight) {
        height = `calc(100vh - ${props.extraHeight})`
    } else if (props.asChild || isChild.value) {
        height = 'auto'
    }
    return {
        height: height,
    }
})

// 处理高度
function setContentHeight() {
   
}

let fieldsetRef = ref(null)

const hasModelTitleComponent = ref(false)

provide('isSlot', true)
const _isSlot = inject('isSlot', false)
if (_isSlot || props.asChild) {
    isChild.value = true
}
defineExpose({
    resetFields: nameList => {
        return formContainerRef?.value.resetFields(nameList)
    },
    validate: _validate,
    clearValidate: _clearValidate,
    validateFields: nameList => {
        return formContainerRef?.value.validateFields(nameList)
    },
    scrollToField: (name, options) => {
        return formContainerRef?.value.scrollToField(name, options)
    },
    resetValidate,
    getSelectedAddressName: name => getSelectedAddressName(name, cityPicker.value),
    footerHeight,
    headerHeight,
    FormPageContainerHeight,
    _name: 'FormPage',
    cityPicker,
    rules,
    hasModelTitleComponent,
})
</script>

<script>
import { ref } from 'vue'
const _hasModelTitleComponent = ref(false)
export default defineComponent({
    data() {
        return {
            _hasModelTitleComponent: _hasModelTitleComponent,
        }
    },
    components: {},
    setTitleComponent(component) {
        this.components['defaultModelTitleComponent'] = component
        _hasModelTitleComponent.value = true
    },
})
</script>
<style lang="less" scoped>
@import './style/formDisabled.less';
@pagewidth: calc (100vw - 239px);

.x-formpage-container {
    position: relative;
    width: 100%;

    .form-container {
        &.form-container-grid {
            display: grid;
        }
    }

    &-form {
        padding: 0 8px 4px;
        box-sizing: border-box;
    }

    &-fixed {
        &-footer {
            border-top: 1px solid #d6d3d3;
            padding-bottom: 10px;
            position: sticky;
            bottom: 0;
            left: 0;
            flex: 1;
            background-color: #fff;
            width: 100%;
            z-index: 8;
        }

        &-header {
            width: 100%;
            z-index: 1;
        }
    }

    .title {
        font-weight: 600;
        color: #7f7f7f;
        font-size: 16px;
        white-space: nowrap;
        text-align: left;
        padding-bottom: 12px;
        padding-top: 8px;
    }

    // 分隔线
    ::v-deep(.ant-divider-horizontal) {
        .ant-divider-inner-text {
            font-weight: 600;
            color: #7f7f7f;
        }
    }
}

.code_img {
    margin-left: 14px;
    height: 32px;
    cursor: pointer;

    .getCaptcha {
        display: block;
        width: 102px;
        height: 32px;
    }
}
</style>

```

## FormPage使用案例

### 案例1：

```vue
<template>
    <div class="p-3">
        <Breadcrumb>
            <a-breadcrumb-item>
                <router-link to="/systemManage/menus">菜单管理</router-link>
            </a-breadcrumb-item>
            <a-breadcrumb-item>{{ isEdit ? '编辑菜单' : '新增菜单' }}</a-breadcrumb-item>
        </Breadcrumb>
        <Spin :spinning="loading">
            <div class="title-wrap">
                <div class="left">{{ isEdit ? '编辑菜单' : '新增菜单' }}</div>
                <div class="btns">
                    <Space>
                        <template v-if="isEdit">
                            <Button type="primary" @click="save('back')">保存</Button>
                        </template>
                        <template v-else>
                            <Button type="primary" @click="save('back')">保存后返回上一页</Button>
                            <Button type="primary" @click="save">保存后留在当前页</Button>
                        </template>
                        <Button @click="goBack()">取消</Button>
                    </Space>
                </div>
            </div>
            <FormPage
                ref="FormPageRef"
                v-model:formData="formData"
                :form-model="formModel"
                :label-col="{ style: { minWidth: '100px' } }"
                verticalGap="12"
            >
                <template #treeSelect>
                    <Row>
                        <Col :span="18">
                            <x-form-item
                                label="上级菜单"
                                label-align="left"
                                :labelCol="{ style: { minWidth: '100px' } }"
                            >
                                <TreeSelect
                                    v-model:value="formData.previousMenuName"
                                    show-search
                                    :fieldNames="{
                                        children: 'children',
                                        title: 'menuName',
                                        key: 'sysMenuId',
                                        value: 'menuName',
                                        label: 'menuName',
                                    }"
                                    :dropdown-style="{ maxHeight: '400px', overflow: 'auto' }"
                                    allow-clear
                                    tree-default-expand-all
                                    :tree-data="menusList"
                                    placeholder="请选择菜单"
                                ></TreeSelect>
                            </x-form-item>
                        </Col>
                    </Row>
                </template>
                <template #text>
                    <Row>
                        <Col :span="18">
                            <x-form-item
                                label="注意"
                                label-align="left"
                                :labelCol="{ style: { minWidth: '100px' } }"
                            >
                                <text class="care" selectable="false" space="false" decode="false">
                                    1、菜单从有效到无效操作：如果当前菜单设置为无效，那么当前菜单和子级菜单全部都会修改为无效；
                                    <br />
                                    2、菜单从无效到有效操作：如果当前菜单设置为有效，那么只有他以及父级菜单才会设置为有效；
                                </text>
                            </x-form-item>
                        </Col>
                    </Row>
                </template>
            </FormPage>
        </Spin>
    </div>
</template>

<script setup>
import { Breadcrumb, Row, Spin, TreeSelect, FormPage, Col, Button, Space } from 'x-widgets'
import useEditMenus from './composables/editMenus/useEditMenus'
import useMenusTree from './composables/editMenus/useMenusTree'
import { useRouter } from 'vue-router'

const router = useRouter()
const { menusList, searchValue, menuDataList } = useMenusTree({
    path: '/systemManage/editMenus',
})

const FormPageRef = ref('')
const { formData, formModel, loading, save, isEdit } = useEditMenus({
    FormPageRef,
    menuDataList,
})

const goBack = () => {
    router.back()
}
</script>

<style lang="less" scoped>
.title-wrap {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    padding: 8px;
    border-top: 1px solid #d4d2d2;
    margin: 24px 0;

    .left {
        font-size: 24px;
        font-weight: bold;
    }
}

.care {
    color: red;
    display: flex;
    flex-direction: column;
    margin-top: 5px;
    font-weight: 400;
}
</style>

```

```js
// useEditMenus.js
import { apis } from '@api'
import { onActivated, onMounted, unref, watch } from 'vue'
import { useRouter } from 'vue-router'
import { message } from 'x-widgets'
import { getRouteQuery } from 'x-utils'
import { getMenuByName, getMenuById } from '@/components/Tree/composables/useMenusTree.js'

export default function useFormModal({ FormPageRef, menuDataList }) {
    const { sysMenuId } = getRouteQuery()
    const router = useRouter()
    const isEdit = ref(false)
    const formData = ref({
        sysMenuId: null,
        previousMenuName: '全部菜单',
        sort: null,
        menuLevel: 1,
        menuLevelStr: '',
        parentId: null,
        pageUrl: null,
        menuStatus: 1,
        menuName: null,
        menuType: 1,
        menuRange: 1,
        menuCode: null,
    })

    const formDataInit = ref({
        sysMenuId: null,
        previousMenuName: '全部菜单',
        sort: null,
        menuLevel: 1,
        menuLevelStr: '',
        parentId: null,
        pageUrl: null,
        menuStatus: 1,
        menuName: null,
        menuType: 1,
        menuRange: 1,
        menuCode: null,
    })

    const formModelOne = {
        title: '',
        divider: false,
        column: [
            {
                slot: 'treeSelect',
            },
            {
                el: 'input',
                label: '序号',
                name: 'sort',
                afterText: '可不填序号，默认升序',
                unitStyle: 'color:red',
                rules: [
                    {
                        pattern: /^\S+$/,
                        message: '不能输入空格',
                    },
                ],
            },
            {
                el: 'text',
                label: '菜单级别',
                name: 'menuLevelStr',
            },
            {
                el: 'input',
                label: '菜单名称',
                name: 'menuName',
                trim: true,
                rules: [
                    { required: true, message: '请输入菜单名称' },
                    {
                        pattern: /^\S+$/,
                        message: '不能输入空格',
                    },
                ],
            },
            {
                el: 'input',
                label: '页面路径',
                name: 'pageUrl',
                trim: true,
                rules: [
                    { required: true, message: '请输入面路径' },
                    {
                        pattern: /^\S+$/,
                        message: '不能输入空格',
                    },
                ],
                // display: computed(() => formData.value.menuType === 1),
            },
            {
                el: 'radio-group',
                label: '状态',
                name: 'menuStatus',
                options: [
                    { label: '有效', value: 1 },
                    { label: '无效', value: 2 },
                ],
            },
            {
                el: 'radio-group',
                label: '菜单类型',
                name: 'menuType',
                options: [
                    { label: '菜单', value: 1 },
                    { label: '目录', value: 2 },
                ],
            },
            {
                el: 'radio-group',
                label: '是否平台菜单',
                name: 'menuRange',
                options: [
                    { label: '否', value: '1' },
                    { label: '是', value: '2' },
                ],
                afterText: '平台菜单表示平台专用的菜单，比如：菜单管理、字典管理、租户管理等',
                unitStyle: 'color:red',
            },
            {
                el: 'input',
                label: '菜单编码',
                name: 'menuCode',
            },
            {
                el: 'input',
                label: '菜单图标',
                name: 'menuIconCode',
                selfProps: {
                    extra: '提示：图标值来源于自定义的阿里图标库',
                },
            },
            {
                el: 'input',
                label: '模块类型',
                name: 'moduleType',
                selfProps: {
                    extra: '提示：用于使用多功能登录入口时，保持当前菜单列表，入口参考1=组织管理，2=粤农财，3=资产管理，4=合同管理，5=监督预警，6=监督管理，7=资产一张图',
                },
            },
            {
                slot: 'text',
            },
        ],
    }

    const formModel = ref([formModelOne])

    const loading = ref(false)
    async function getProjectDetail(sysMenuId) {
        try {
            loading.value = true
            let { data } = await apis.systemManage.getMenuById({
                sysMenuId: sysMenuId,
            })
            Object.keys(data).forEach(key => {
                if (formData.value.hasOwnProperty(key)) {
                    formData.value[key] = data[key]
                }
            })
            formData.value.menuStatus = data.menuStatus ? 1 : 2
            formData.value.menuType = data.menuType ? 1 : 2
            loading.value = false
        } catch (error) {
            loading.value = false
        }
    }
    function save(tag) {
        FormPageRef.value
            .validate()
            .then(async res => {
                try {
                    loading.value = true
                    const params = {
                        ...formData.value,
                        menuStatus: formData.value.menuStatus === 1 ? true : false,
                        menuType: formData.value.menuType === 1 ? 1 : 0,
                        parentId:
                            getMenuByName(formData.value.previousMenuName, menuDataList.value)
                                ?.key || 0,
                        moduleType: formData.value.moduleType
                            ? Number(formData.value.moduleType)
                            : null,
                    }
                    let api
                    if (sysMenuId) {
                        api = apis.systemManage.updateMenu
                    } else {
                        api = apis.systemManage.saveMenu
                    }
                    let { data } = await api(params)
                    if (data) {
                        message.success('保存成功')
                        if (tag === 'back') {
                            router.back()
                        }
                    }
                    loading.value = false
                } catch (error) {
                    loading.value = false
                }
            })
            .catch(err => {})
    }
    const levelNameMap = {
        1: '一级菜单',
        2: '二级菜单',
        3: '三级菜单',
        4: '四级菜单',
        5: '五级菜单',
    }
    watch(
        () => formData.value.previousMenuName,
        val => {
            if (val) {
                const level = getMenuByName(val, menuDataList.value)?.menuLevel + 1
                const name = levelNameMap[level] || level
                formData.value.menuLevel = level
                formData.value.menuLevelStr = name
            } else {
                formData.value.menuLevel = 1
                formData.value.menuLevelStr = '一级菜单'
            }
        },
    )
    onActivated(async () => {
        const { sysMenuId, currentSysMenuId, menuName, menuLevel, previousMenuName } =
            getRouteQuery()
        isEdit.value = sysMenuId ? true : false
        FormPageRef.value.resetFields()
        if (isEdit.value) {
            getProjectDetail(sysMenuId)
            formData.value.menuName = menuName
            formData.value.sysMenuId = sysMenuId
            formData.value.previousMenuName = previousMenuName
            formData.value.menuLevel = menuLevel
        } else {
            Object.assign(formData.value, formDataInit.value)
            formData.value.previousMenuName = getMenuById(
                currentSysMenuId,
                menuDataList.value,
            )?.title
            formData.value.menuLevelStr = levelNameMap[formData.value.menuLevel]
        }
    })
    return {
        formData,
        formModel,
        loading,
        save,
        sysMenuId,
        isEdit,
    }
}

```

### 案例二

```vue
<template>
        <FormPage
            ref="FormPageRef"
            v-model:formData="formData"
            :form-model="formModel"
            :loading="loading"
            :label-col="{ style: { minWidth: '100px' } }"
        ></FormPage>
</template>

<script setup>
import { ref } from 'vue'
import FormPage from '../../FormPage/FormPage.vue'
import useEditFarmerInfo from './composables/useEditFarmerInfo.js'

const FormPageRef = ref(null)

const { formData, formModel, loading, save } = useEditFarmerInfo({
    FormPageRef,
})
</script>

<style scoped>
.btns {
    text-align: center;
}
</style>

```

```js
// useFormModal.js

import { apis } from '@api'
import { Modal as AModal, message as AMessage } from 'ant-design-vue'
import {
    idType,
    sex,
    relation,
    accountType,
    maritalStatus,
    memberRemark,
    yesNo,
    dataSource,
} from '@/config/selections'
import { ref } from 'vue'
import { useRoute } from 'vue-router'
import { isValidIdNumber } from '@/utils/validate'

export default function useFormModal({ list }, successCallback, autoFillForm) {
    const route = useRoute()
    const isEdit = computed(() => route.query?.farmerAttributesId)

    const showModal = ref(false),
        modalTitle = ref('添加户内成员')

    const formData = ref({
        memberName: null,
        memberCardType: null,
        memberCardNumber: null,
        sex: null,
        relationship: null,
        householdRegistrationType: null,
        phone: null,
        maritalStatus: null,
        organizationMember: null,
        memberType: null,
        memberRemark: null,
        dataSource: null,
        remark: null,
    })

    // 表单字段
    const column = {
        title: '',
        column: [
            {
                el: 'input',
                label: '姓名',
                name: 'memberName',
                colspan: 8,
                rules: [{ required: true, message: '请输入姓名' }],
            },
            {
                el: 'select',
                label: '证件类型',
                name: 'memberCardType',
                colspan: 8,
                options: idType,
                event: {
                    change: val => {
                        formData.value.memberCardNumber = ''
                    },
                },
                rules: [{ required: true, message: '请选择证件类型' }],
            },
            {
                el: 'input',
                label: '证件号码',
                name: 'memberCardNumber',
                colspan: 8,
                rules: [
                    {
                        required: true,
                        validator: (rule, value) => {
                            if (!value) return Promise.reject('请输入证件号码')
                            if (formData.value.memberCardType === '01' && !isValidIdNumber(value)) {
                                return Promise.reject('请输入正确的身份证号码')
                            }
                            return Promise.resolve()
                        },
                    },
                ],
            },
            {
                el: 'select',
                label: '性别',
                name: 'sex',
                colspan: 8,
                options: sex,
                rules: [{ required: true, message: '请选择性别' }],
            },
            {
                el: 'select',
                label: '与户主关系',
                name: 'relationship',
                colspan: 8,
                options: relation,
                rules: [{ required: true, message: '请选择与户主关系' }],
            },
            {
                el: 'select',
                label: '户口类型',
                name: 'householdRegistrationType',
                colspan: 8,
                options: accountType,
                rules: [{ required: true, message: '请选择户口类型' }],
            },
            {
                el: 'input',
                label: '联系电话',
                name: 'phone',
                colspan: 8,
            },
            {
                el: 'select',
                label: '婚姻状况',
                name: 'maritalStatus',
                colspan: 8,
                options: maritalStatus,
            },
            {
                el: 'select',
                label: '是否本集体经济组织成员',
                name: 'organizationMember',
                autoLabelHeight: true,
                attributes: {
                    labelCol: {
                        style: {
                            maxWidth: '120px',
                        },
                    },
                },
                props: {
                    style: {
                        maxWidth: '270px',
                    },
                },
                colspan: 8,
                options: yesNo,
                rules: [{ required: true, message: '请选择是否本集体经济组织成员' }],
            },
            {
                el: 'select',
                label: '成员备注',
                name: 'memberType',
                colspan: 8,
                options: memberRemark,
            },
            {
                el: 'input',
                label: '成员备注说明',
                name: 'memberRemark',
                colspan: 8,
            },
            {
                el: 'select',
                label: '数据来源',
                name: 'dataSource',
                colspan: 8,
                options: dataSource,
            },
            {
                el: 'input',
                label: '备注',
                name: 'remark',
                colspan: 8,
            },
        ],
    }
    const formModel = ref([column])

    function validate(params) {
        const owner = list.value.find(item => item.relationship === '10')
        const currentNer = list.value.find(
            item =>
                (item.tempId && item.tempId === params.tempId) ||
                (item.farmerMemberId && item.farmerMemberId === params.farmerMemberId),
        )
        if (owner?.tempId !== currentNer?.tempId && currentNer.relationship === '10') {
            AMessage.error('已存在户主信息, 不允许再添加')
            return false
        }
        if (
            list.value.find(
                item =>
                    item.memberCardType === params.memberCardType &&
                    item.memberCardNumber === params.memberCardNumber &&
                    item.tempId !== params.tempId,
            )
        ) {
            AMessage.error('已存在相同的证件号码')
            return false
        }
        return true
    }
    // 确认新增
    async function confirmAdd(params) {
        try {
            if (isEdit.value) {
                if (modalTitle.value === '添加户内成员') {
                    //编辑页面的新增
                    await apis.homesteadAllocation.saveMember({
                        ...params,
                        farmerAttributesId: route.query.farmerAttributesId,
                    })
                } else {
                    //编辑页面的编辑
                    await apis.homesteadAllocation.updateMember({
                        ...params,
                        farmerAttributesId: route.query.farmerAttributesId,
                    })
                }
                AMessage.success('操作成功')
                successCallback()
            } else {
                //首次新增时
                if (modalTitle.value === '添加户内成员') {
                    //新增, 本地缓存表格数据
                    const _params = JSON.parse(JSON.stringify({ ...params, tempId: Date.now() }))
                    if (!validate(_params)) return
                    list.value.push(_params)
                } else {
                    //编辑
                    const index = list.value.findIndex(item => item.tempId === params.tempId)
                    const _params = JSON.parse(JSON.stringify(params))
                    if (!validate(_params)) return
                    list.value[index] = Object.assign(list.value[index], _params)
                }
            }
            showModal.value = false
            autoFillForm()
        } catch (error) {
            throw error
        }
    }

    function editForm(params) {
        Object.assign(formData.value, params)
    }

    const loading = ref(false)
    function getDetail(record) {
        loading.value = true
        apis.homesteadAllocation
            .getMemberInfo({
                farmerMemberId: record.farmerMemberId,
            })
            .then(res => {
                editForm(res.data)
                loading.value = false
            })
    }

    function fillForm(_record) {
        if (isEdit.value) {
            getDetail(_record)
        } else {
            editForm(_record)
        }
    }

    const cancelText = ref('取消')
    const showConfirmBtn = ref(true)
    const modalDisabled = ref(false)

    function handleShowModal(type, record) {
        const _record = (record && JSON.parse(JSON.stringify(record))) || {}
        modalDisabled.value = false
        showConfirmBtn.value = true
        cancelText.value = '取消'
        if (type === 'edit') {
            modalTitle.value = '编辑户内成员'
            fillForm(_record)
        } else if (type === 'add') {
            modalTitle.value = '添加户内成员'
        } else if (type === 'detail') {
            fillForm(_record)
            modalTitle.value = '户内成员详情'
            modalDisabled.value = true
            showConfirmBtn.value = false
            cancelText.value = '关闭'
        }
        showModal.value = true
    }
    return {
        formData,
        formModel,
        showModal,
        modalTitle,
        confirmAdd,
        handleShowModal,
        modalDisabled,
        showConfirmBtn,
        cancelText,
        editForm,
        loading,
    }
}

```

