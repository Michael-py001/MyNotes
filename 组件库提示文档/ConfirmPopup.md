## ConfirmPopup组件使用说明文档

> 该组件用于弹出确认窗口，下方有确认和取消两个按钮，中间可自定义内容。

## 源码

以下是组件库的源代码：

```vue
<!-- 
    @description 确认弹窗组件
		@name ConfirmPopup
    @author WuShiLi
 -->
<template>
    <div class="li-ConfirmPopup">
        <Modal
            :open="show"
            :centered="true"
            :closable="closable"
            :mask-closable="false"
            :width="width"
            class="li-ConfirmPopup-container"
            destroyOnClose
            @cancel="clickCancel"
            :showFullScreenIcon="showFullScreenIcon"
            v-bind="$attrs"
            :aria-hidden="!show"
        >
            <template #title>
                <slot name="title">
                    <span class="li-ConfirmPopup-title">
                        {{ title }}
                    </span>
                </slot>
            </template>
            <slot></slot>
            <template #footer>
                <slot name="footer">
                    <Button
                        v-if="showCancelBtn"
                        :disabled="okBtnLoading"
                        type="primary"
                        ghost
                        @click="clickCancel"
                        v-bind="{ ...$attrs.cancelButtonProps }"
                    >
                        {{ cancelText }}
                    </Button>
                    <Button
                        v-if="showConfirmBtn"
                        type="primary"
                        :disabled="okBtnLoading || disabled"
                        :loading="okBtnLoading"
                        @click="clickConfirm"
                        v-bind="{ ...$attrs.okButtonProps }"
                    >
                        {{ confirmText }}
                    </Button>
                </slot>
            </template>
        </Modal>
    </div>
</template>

<script setup>
import { Modal, Button } from '../../index'
const emit = defineEmits(['update:formData', 'confirm', 'cancel', 'update:show'])
defineOptions({
    name: 'XConfirmPopup',
})
const props = defineProps({
    title: {
        type: String,
        default: '标题',
    },
    show: {
        type: Boolean,
        default: false,
    },
    // 弹窗宽度
    width: {
        type: [String, Number],
        default: '500px',
    },
    // 重置按钮文字，传空则不显示按钮
    cancelText: {
        type: String,
        default: '取消',
    },
    // 搜索按钮文字，传空则不显示按钮
    confirmText: {
        type: String,
        default: '确认',
    },
    // 是否显示取消按钮
    showCancelBtn: {
        type: Boolean,
        default: true,
    },
    // 是否显示确认按钮
    showConfirmBtn: {
        type: Boolean,
        default: true,
    },
    //确认按钮loading
    okBtnLoading: {
        type: Boolean,
        default: false,
    },
    // 是否显示右上角关闭按钮
    closable: {
        type: Boolean,
        default: false,
    },
    showFullScreenIcon: {
        type: Boolean,
        default: false,
    },
    // 是否禁用
    disabled: {
        type: Boolean,
        default: false,
    },
})

// 确认回调
function clickConfirm() {
    emit('confirm')
}

// 取消回调
function clickCancel() {
    emit('update:show', false)
    emit('cancel')
}
</script>

<style lang="less">
.content {
    overflow-y: auto;
}
</style>

```

## 使用案例

```vue
<script setup>
import Basic from './index.vue';
import Demo2 from './demo2.vue';
import Demo3 from './demo3.vue';
import Demo4 from './demo4.vue';
import Demo5 from './demo5.vue';
</script>
```

### 基础用法

```vue
<template>
    <a-button @click="showPopup = true" type="primary">打开弹窗</a-button>
    <ConfirmPopup
        :show="showPopup"
        modal-title="温馨提示"
        cancel-text="再想想"
        confirm-text="确认关闭"
        @confirm="showPopup = false"
    >
        <div class="flex-row">您确认要关闭弹窗吗？</div>
    </ConfirmPopup>
</template>

<script setup>
import { ref } from 'vue'
import { ConfirmPopup } from 'x-widgets'

const showPopup = ref(false)
</script>
```

### 底部按钮显示控制

```vue
<template>
    <ConfirmPopup
        v-model:show="showPopup"
        modal-title="温馨提示"
        modal-width="600px"
        :showConfirmBtn="false"
        @confirm="showPopup = false"
    >
        <div class="flex-row">这是一则温馨提示</div>
    </ConfirmPopup>
</template>
```

### 隐藏确认按钮

```vue
<template>
    <ConfirmPopup
        v-model:show="showPopup"
        modal-title="温馨提示"
        modal-width="600px"
        :showConfirmBtn="false"
        @confirm="showPopup = false"
    >
        <div class="flex-row">这是一则温馨提示</div>
    </ConfirmPopup>
</template>
```

### 隐藏所有按钮

```vue
<template>
    <ConfirmPopup
        v-model:show="showPopup2"
        modal-title="温馨提示"
        modal-width="600px"
        :footer="false"
        :closable="true"
        @confirm="showPopup2 = false"
    >
        <div class="flex-row">这是一则温馨提示</div>
    </ConfirmPopup>
</template>
```

### 按钮 loading 状态

```vue
<template>
    <ConfirmPopup
        v-model:show="showPopup"
        cancel-text="再想想"
        confirm-text="确认关闭"
        modalWidth="800px"
        @confirm="showPopup = false"
    >
        <template #title>
            <div>⚠️警告</div>
        </template>
        <div class="flex-row">您确认要关闭弹窗吗？</div>
    </ConfirmPopup>
</template>
```

### 自定义弹窗宽度

```vue
<template>
    <ConfirmPopup
        v-model:show="showPopup"
        modal-title="温馨提示"
        cancel-text="再想想"
        confirm-text="确认关闭"
        modalWidth="800px"
        @confirm="showPopup = false"
    >
        <div class="flex-row">您确认要关闭弹窗吗？</div>
    </ConfirmPopup>
</template>
```

### 插槽: 自定义标题

```vue
<template>
    <ConfirmPopup
        v-model:show="showPopup"
        cancel-text="再想想"
        confirm-text="确认关闭"
        modalWidth="800px"
        @confirm="showPopup = false"
    >
        <template #title>
            <div>⚠️警告</div>
        </template>
        <div class="flex-row">您确认要关闭弹窗吗？</div>
    </ConfirmPopup>
</template>
```

### ConfirmPopup 属性 Props

| 参数               | 说明                           | 类型               | 默认值  |
| ------------------ | ------------------------------ | ------------------ | ------- |
| modalTitle         | 标题                           | _string_           | ` 标题` |
| show(v-model)      | 是否显示弹窗                   | _boolean_          | `false` |
| modalWidth         | 弹窗宽度                       | _string \| number_ | `500px` |
| cancelText         | 重置按钮文字，传空则不显示按钮 | _string_           | `取消`  |
| confirmText        | 搜索按钮文字，传空则不显示按钮 | _string_           | `确认`  |
| showCancelBtn      | 是否显示取消按钮               | _boolean_          | `true`  |
| showConfirmBtn     | 是否显示确认按钮               | _boolean_          | `true`  |
| okBtnLoading       | 确认按钮 loading               | _boolean_          | `false` |
| closable           | 是否显示右上角关闭按钮         | _boolean_          | `false` |
| showFullScreenIcon | 是否显示全屏按钮               | _boolean_          | `false` |
| disabled           | 是否禁用                       | _boolean_          | `false` |

## 实际项目中使用的案例

```tsx
import { defineComponent, ref, nextTick } from 'vue'
import type { PropType } from 'vue'
import { ConfirmPopup } from 'x-widgets'
import type { fileProps } from './types/props'
import style from './style/index.module.less'
import SwiperFilesPreview from './SwiperFilesPreview'

/**
 * @description: 弹窗形式图片/文档/视频预览组件
 * @author WuShiLi
 */

export default defineComponent({
    name: 'SwiperModal',
    props: {
        // 文件列表
        fileList: {
            type: Array as PropType<fileProps[]>,
            default: () => [],
        },
        title: {
            type: String,
            default: '文件预览',
        },
        open: {
            type: Boolean,
            default: false,
        },
        showName: {
            type: Boolean,
            default: true,
        },
    },
    emits: ['update:open', 'update:fileList'],
    setup(props, { attrs, slots, emit, expose }) {
        const { title } = props
        const FilesPreviewRef = ref(null)
        function goTo(index) {
            nextTick(() => {
                FilesPreviewRef.value && FilesPreviewRef.value.goTo(index)
            })
        }
        expose({
            goTo,
        })
        return () => {
            return (
                <ConfirmPopup
                    show={props.open}
                    title={title}
                    closable
                    width="1000px"
                    footer={null}
                    onCancel={() => emit('update:open', false)}
                    showFullScreenIcon={false}
                >
                    <div class={style._SwiperModalWrap}>
                        <SwiperFilesPreview
                            showName={props.showName}
                            v-model={[props.fileList, 'fileList']}
                            ref={FilesPreviewRef}
                            {...attrs}
                        ></SwiperFilesPreview>
                    </div>
                </ConfirmPopup>
            )
        }
    },
})

```

```vue
<!-- 
    @description: 数据应用其它账套
    @author WuShiLi
 -->
<template>
    <div class="ErrorTableModal">
        <ConfirmPopup
            :show="open"
            title="数据应用其它账套"
            @confirm="handleOk"
            width="60vw"
            closable
            :showFullScreenIcon="false"
            @cancel="$emit('update:open', false)"
        >
            <div class="content">
                <div class="error_tip">
                    <slot name="title"></slot>
                </div>
                <div>
                    <Form layout="vertical">
                        <FormItem
                            label="请选择账套名称："
                            :rules="[{ required: true, message: '请选择账套名称' }]"
                        >
                            <Col>
                                <AccountCheckedTree
                                    :api="accountApi"
                                    :ref="ref => (accountTreeRef = ref)"
                                    :activeMenuItem="activeMenuItem"
                                    :year="year"
                                ></AccountCheckedTree>
                            </Col>
                        </FormItem>
                    </Form>
                </div>
            </div>
        </ConfirmPopup>
    </div>
</template>

<script setup>
import AccountCheckedTree from '@/components/Tree/AccountCheckedTree.vue'
import { ConfirmPopup, message } from 'x-widgets'
import { Modal, Table, FormItem, Col, Row, Form } from 'ant-design-vue'
import { apis } from '@api'
import { watch, nextTick } from 'vue'
const emit = defineEmits(['update:open', 'submit'])
const props = defineProps({
    // 账套树api
    accountApi: {
        type: Function,
        required: true,
    },
    open: {
        type: Boolean,
        default: false,
    },
    // 选中的账套
    activeMenuItem: {
        type: [String, Object],
        default: () => {},
    },
    // 选中的科目
    selectedItems: {
        type: [String, Object],
        default: () => {},
    },
    year: {
        type: [String, Number],
        default: '',
    },
})
const accountTreeRef = ref(null)
function handleOk() {
    if (accountTreeRef.value.checkedItems.length) {
        const checkedKeyData = accountTreeRef.value.checkedItems
            .filter(item => item.baseAccountBookId !== '0')
            .map(el => el.baseAccountBookId)
        emit('submit', checkedKeyData)
    } else {
        message.error('请选择账套')
    }
}

watch(
    () => props.open,
    async newVal => {
        if (newVal) {
            nextTick(async () => {
                if (accountTreeRef.value) {
                    await accountTreeRef.value?.getMenusList()
                    accountTreeRef.value.checkedKeys = []
                    accountTreeRef.value?.setExpendKeys()
                }
            })
        }
    },
)
</script>

<style lang="less" scoped>
.content {
    min-height: 500px;
}
.error_tip {
    display: flex;
    justify-content: space-between;
    margin-bottom: 20px;
    align-items: center;
}
</style>

```

```vue
<ConfirmPopup
            class="stageslModal"
            :show="openStagesDetails"
            title="分期详情"
            width="60vw"
            closable
            showFullScreenIcon
            @cancel="handleDetailCancel"
        >
            <div v-if="isChange" class="mb-[5px]">
                说明：符合以下条件之一的分期不允许变更：1、已履约的分期；2、部分履约；3、已逾期且缴纳了部分金额；4、已经产生了会计凭证的分期；
            </div>
            <StagesDetailsPage
                ref="stagesDetails"
                :tableData="tableData"
                :isEdit="true"
                v-model:formData="innerFormData"
            ></StagesDetailsPage>
            <template #footer>
                <div class="footer">
                    <x-flex justify="end" gap="10">
                        <Button type="primary" ghost @click="handleDetailCancel">取消</Button>
                        <Button type="primary" ghost @click="handleBack()">上一步</Button>
                        <Button type="primary" @click="handleSave()">保存</Button>
                    </x-flex>
                </div>
            </template>
        </ConfirmPopup>
```

