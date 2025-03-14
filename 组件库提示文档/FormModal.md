## FormModal 组件说明文档

组件概述：FormModal 是一个弹窗式的表单组件，是由 FormPage 和 ConfirmPopup 两个组件结合而成，用于页面中弹窗式的表单需求。

```vue
<!-- 
@Description: 弹窗表单组件
@Name: FormModal
@Author: WuShiLi
-->
<template>
<div class="FormModal">
  <ConfirmPopup
                :width="modalWidth"
                :title="modalTitle"
                :show="show"
                :cancel-text="cancelText"
                :confirm-text="confirmText"
                :ok-btn-loading="okBtnLoading"
                :show-cancel-btn="showCancelBtn"
                :show-confirm-btn="showConfirmBtn"
                :show-footer="showFooter"
                v-bind="$attrs"
                :show-full-screen-icon="showFullScreenIcon"
                :disabled="$attrs.disabled || $attrs.loading"
                @confirm="clickConfirm"
                @cancel="clickCancel"
                >
    <template #title>
      <slot name="title"></slot>
</template>
<FormPage
          ref="FormPageRef"
          v-model:formData="innerFormData"
          :form-model="formModel"
          :wrapperCol="wrapperCol"
          :labelCol="labelCol"
          :labelAlign="labelAlign"
          :fieldsetStyle="fieldsetStyle"
          :autocomplete="autocomplete"
          asChild
          v-bind="$attrs"
          :gutter="gutter"
          >
  <template
            v-for="(name, index) in Object.keys(slot)"
            #[name]="scopeData"
            :key="index"
            >
<slot :name="name" v-bind="scopeData" :key="index"></slot>
  </template>
</FormPage>
<template #footer>
<slot name="modalFooter"></slot>
</template>
</ConfirmPopup>
</div>
</template>

<script setup>
  import FormPage from '../FormPage/FormPage.vue'
  import ConfirmPopup from '../ConfirmPopup/ConfirmPopup.vue'
  import { computed, useSlots, unref } from 'vue'

  const slot = useSlots()
  defineOptions({
    name: 'XFormModal',
  })
  const emit = defineEmits(['update:formData', 'confirm', 'cancel', 'update:show'])

  // FormModal的所有props
  const props = defineProps({
    // 弹窗标题
    modalTitle: {
      type: String,
      default: '标题',
    },
    // 是否显示
    show: {
      type: Boolean,
      default: false,
    },
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
      default: 'right', //'left' | 'right'
    },
    // 弹窗宽度
    modalWidth: {
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
    //表单间隔
    gutter: {
      type: Array,
      default: () => [12, 0],
    },
    //表单确按钮loading
    okBtnLoading: {
      type: Boolean,
      default: false,
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
    //字段colspan
    colspan: {
      type: Number,
      default: 24,
    },
    // 字段垂直间隔
    verticalGap: {
      type: [String, Number],
      default: '24',
    },
    //是否显示footer
    showFooter: {
      type: Boolean,
      default: true,
    },
    // 表单布局
    layout: {
      type: String,
      default: 'horizontal', //'horizontal' | 'vertical' | 'inline'
    },
    autoClose: {
      type: Boolean,
      default: true,
    },
    fieldsetStyle: {
      type: [Object, String],
      default: () => {
        return {}
      },
    },
    //是否显示全屏
    showFullScreenIcon: {
      type: Boolean,
      default: true,
    },
    autocomplete: {
      type: String,
      default: 'on',
    },
  })

  const innerFormData = computed({
    get: () => props.formData,
  })

  const FormPageRef = ref(null)

  // 确认回调
  function clickConfirm() {
    FormPageRef.value
      .validate()
      .then(res => {
      emit('confirm', unref(innerFormData))
    })
      .catch(error => {
      /* eslint-disable */
      console.warn('error:', error)
    })
  }

  // 取消回调
  function clickCancel() {
    if (props.autoClose) {
      FormPageRef.value.resetFields()
      emit('update:show', false)
      emit('cancel', false)
    } else {
      emit('cancel', false)
    }
  }

  defineExpose({
    resetFields: nameList => FormPageRef.value.resetFields(nameList),
    validateFields: nameList => FormPageRef.value.validateFields(nameList),
    validate: nameList => FormPageRef.value.validate(nameList),
    FormPageRef,
    clickConfirm,
    clickCancel,
  })
</script>

<style lang="less" scoped>
  .FormModal-container {
    .FormModal-footer {
      display: flex;
      justify-content: space-around;
      align-items: center;
      padding: 0 14%;
      .ant-btn {
        width: 130px;
        height: 36px;
      }
    }
  }
  .title {
    font-size: 22px;
    font-weight: bold;
    padding-right: 20px;
  }
  .second-title {
    font-size: 16px;
  }
  .title-wrap {
    display: flex;
    justify-content: flex-start;
    align-items: center;
  }
  .input-line {
    display: flex;
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

## 实际项目中的使用

```vue
<template>
 <!-- 记账信息弹窗 -->
        <FormModal
            ref="formModalRef"
            v-model:formData="modalFormData"
            v-model:show="showModal"
            :formModel="modal_formModel"
            :modal-title="modalTitle"
            layout="vertical"
            modal-width="1400px"
            extraHeight="220"
            verticalGap="0"
            confirm-text="保存"
            labelAlign="right"
            :labelCol="{ style: { minWidth: '100px' } }"
            :loading="modal_loading"
            closable
            :autoClose="false"
            showFullScreenIcon
            :fieldsetStyle="fieldsetStyle"
            @confirm="confirmAdd"
            @cancel="confirmClose"
        >
            <template #modelTitle="{ title }">
                <template v-if="title === '收入信息' || title === '支出信息'">
                    <FormPageModelTitle :title="title">
                        <template #right>
                            <div class="flex gap-3">
                                <x-button type="primary" size="small" @click="confrimLastRecord">
                                    复用上次记账内容
                                </x-button>
                                <x-tooltip placement="bottomLeft">
                                    <template #title>
                                        <p>复用上次记账内容说明</p>
                                        <p class="flex gap-2">
                                            <span>1</span>
                                            <span>
                                                为了提高出纳记账效率，可根据科目，系统自动回显最近一条收入内容
                                            </span>
                                        </p>
                                        <p class="flex gap-2">
                                            <span>2</span>
                                            <span>回显内容：</span>
                                        </p>
                                        <ul class="pl-4" v-if="modalTitle.includes('收入')">
                                            <li>* 科目</li>
                                            <li>* 关联业务</li>
                                            <li>* 收入类型</li>
                                            <li>* 互转科目（银行存款科目或库存现金科目）</li>
                                            <li>* 结算方式</li>
                                            <li>* 结算号</li>
                                            <li>* 本单位经手人</li>
                                            <li>* 付款方单位/人</li>
                                            <li>* 付款方账号</li>
                                            <li>* 付款方银行名称</li>
                                        </ul>
                                        <ul class="pl-4" v-else>
                                            <li>* 科目</li>
                                            <li>* 关联业务</li>
                                            <li>* 支出类型</li>
                                            <li>* 互转科目（银行存款科目或库存现金科目）</li>
                                            <li>* 结算方式</li>
                                            <li>* 结算号</li>
                                            <li>* 用途</li>
                                            <li>* 本单位经手人</li>
                                            <li>* 审批人</li>
                                            <li>* 收款单位/人</li>
                                            <li>* 收款账号</li>
                                            <li>* 收款银行名称</li>
                                            <li>* 证明人</li>
                                            <li>* 验收人</li>
                                        </ul>
                                    </template>
                                    <QuestionCircleOutlined />
                                </x-tooltip>
                            </div>
                        </template>
                    </FormPageModelTitle>
                </template>
            </template>
            <template #finContractStagingIdList>
                <SelectButton
                    ref="SelectButtonRef"
                    @submit="selectContract"
                    :currentAmountType="currentAmountType"
                    :disabled="hasFinVoucherId"
                    :modalFormData="modalFormData"
                ></SelectButton>
            </template>
            <template #billUrlList>
                <FileListUpload
                    btn-text="上传原始凭证"
                    :maxCount="99"
                    v-model:fileList="modalFormData.billUrlList"
                    accept=".png, .jpg, .gif, .bmp , .doc, .docx, .xls, .xlsx, .pdf"
                    folderName="cashier"
                    :disabled="hasVoucher"
                >
                    <template #tip>
                        <div class="text-gray-500 mt-2">
                            记录或证明经济业务发生或完成的原始凭证，数量不限制，1个≤100M，支持png/jpg/gif/bmp/doc/docx/xls/xlsx/pdf格式，附件名称不能包含以下特殊符号：
                            \ / : * ? " &lt; &gt; |
                        </div>
                    </template>
                </FileListUpload>
            </template>
            <template #otherBillUrlList>
                <FileListUpload
                    btn-text="上传原始凭证"
                    :maxCount="99"
                    v-model:fileList="modalFormData.otherBillUrlList"
                    accept=".png, .jpg, .gif, .bmp , .doc, .docx, .xls, .xlsx, .pdf"
                    folderName="cashier"
                    :disabled="hasVoucher"
                >
                    <template #tip>
                        <div class="text-gray-500 mt-2">
                            记录或证明经济业务发生或完成的原始凭证，数量不限制，1个≤100M，支持png/jpg/gif/bmp/doc/docx/xls/xlsx/pdf格式，附件名称不能包含以下特殊符号：
                            \ / : * ? " &lt; &gt; |
                        </div>
                    </template>
                </FileListUpload>
            </template>
            <template #modalFooter>
                <div class="flex justify-evenly">
                    <x-button
                        type="primary"
                        @click="submit(false)"
                        :loading="modal_loading"
                        v-if="
                            modalFormData.temporarySave === true ||
                            modalFormData.temporarySave === null
                        "
                    >
                        暂存
                    </x-button>
                    <x-button type="primary" @click="confirmSubmit" :loading="modal_loading">
                        确定
                    </x-button>
                    <x-button
                        type="primary"
                        @click="confirmSubmitAndNext"
                        :loading="modal_loading"
                        v-if="currentAddType"
                    >
                        确定并下一个
                    </x-button>
                <x-button @click="confirmClose" :loading="modal_loading">取消</x-button>
                </div>
            </template>
          </FormModal>
        </template>

<script setup>
  import { SearchForm, FormModal, Table, message } from 'x-widgets'
  import useFormModal from './composables/useFormModal'
  
  const {
    formData: modalFormData,
    formModel: modal_formModel,
    showModal,
    modalTitle,
    confirmAdd,
    confirmClose,
    handleShowModal,
    loading: modal_loading,
    clickVoucher,
    voucherUploadModalOpen,
    currentRecord,
    submit,
    selectContract,
    currentAmountType,
    currentAddType,
    hasFinVoucherId,
    hasVoucher,
    confrimLastRecord,
} = useFormModal({
    activeMenuItem,
    successCallback: async () => {
        serialNoSortWay.value = 'descend'
        bookkeepingDateSortWay.value = 0
        return await RefreshList()
    },
    formModalRef,
    SelectButtonRef,
})
 </script>
```

使用时需要配置column和formData:

```js
// useFormModal.js
import { apis } from '@api'
import { Modal, message } from 'x-widgets'
import { computed, nextTick, reactive, ref, watch } from 'vue'
import {
    formatPriceToFixed,
    useForm,
    getNodeById,
    getNodeByKeyDeep,
    forEachDeep,
    sleep,
    findTargetIdToLabel,
} from 'x-utils'
import { getBookkeepingInfo } from '@/components/IncomeDetailModal/composables/useIncomeDetailModal.js'
import dayjs from 'dayjs'
import _ from 'lodash-es'
import { filterOption, getDateWithAccountPeriod, compareObjects } from '@/utils'
import {
    currentAccount,
    accountBasicParams,
    getSelections,
    clearSelections,
    querySelections,
    transformParams,
    transformJSONStringToObject,
    transformListToIds,
    deleteCustomItem,
    getFinalShowValue,
} from '@/shared'

import useSelections from './useSelections.js'

export default function useFormModal({
    activeMenuItem,
    successCallback,
    formModalRef,
    SelectButtonRef,
}) {
    const showModal = ref(false),
        modalTitle = ref('收入新增')

    const _formData = reactive({
        ...accountBasicParams(),
        finBookkeepingId: null, //记账id
        baseAccountSubjectId: null, //科目id
        serialNo: null, //流水号
        bookkeepingDate: getDateWithAccountPeriod()?.format('YYYY-MM-DD'), //记账日期 默认当天
        inAmount: null, //收入
        outAmount: null, //支出金额
        inAmountDetail: undefined, //收入类型
        inAmountSaveFlag: false, //保存收入类型常用
        outAmountDetail: undefined, //支出类型
        outAmountSaveFlag: false, //保存支出类型常用
        summary: undefined, //摘要
        summarySaveFlag: false, //保存摘要常用
        originVoucherList: [], //原始凭证来源
        originVoucherSaveFlag: false, //保存原始凭证来源常用
        balance: null, //余额
        billCount: null, //单据数量
        settlementMode: null, //结算方式
        settlementNo: null, //结算号
        handlerList: [], //本单位经手人
        handlerSaveFlag: false, //保存本单位经手人常用
        billerList: [], //付款方单位/人
        billerSaveFlag: false, //保存付款方单位/人常用
        payeeList: [], //收款人
        payeeSaveFlag: false, //保存收款人常用
        //原始凭证上传附件
        billUrlList: null,
        // 支出
        purpose: null, //用途
        auditorList: [], //审批人
        auditorSaveFlag: false, //保存审批人常用
        certifierList: [], //证明人
        certifierSaveFlag: false, //保存证明人常用
        accepterList: [], //验收人
        accepterSaveFlag: false, //保存验收人常用
        finOriginVoucherId: null, //原始凭证id
        temporarySave: null, //是否暂存 0=否 1=是

        // 收入 / 支出 共用
        payeeAccountType: null, //付款方账户类型
        payeeAccountNo: null, //付款方银行账户
        payeeBank: null, //付款方银行名称
        originSubjectId: null, //原科目id

        payeeAccountNoSaveFlag: false, //保存付款方银行账户常用
        payeeBankSaveFlag: false, //保存付款方银行名称常用
        otherBillUrlList: null, //其它原始凭证

        finContractId: null,
        finContractStagingIdList: [], // 合同分期id列表
        contractName: null, //合同名称
        contractSerialNumber: null, //合同编号
        periods: null, //合同期数
        totalNumberOfInstallments: null, //总期数
        performanceAmount: null, //履约金额
        actualPerformanceAmount: null, //实际履约金额

        finVoucherId: null, // 凭证id
        bookkeepingType: null, //记账类型 0-互转流水, 1-红冲流水
        voucherStatus: null, //凭证状态，0-暂存，1-未提交，2-审核中，3-已驳回，4-待记账，5-已记账

        performanceDateType: null, //履约日期类型
    })

    const { formData, editForm, resetForm } = useForm(_formData)
    const oldFormData = ref(null)

    const {
        inAmountDetail,
        outAmountDetail,
        abstractList,
        originVoucherList,
        handlerList,
        billerList,
        payeeList,
        auditorList,
        certifierList,
        accepterList,
        settlementMode,
        payeeAccountNo,
        payeeBank,
        handleOutcome,
        hanldeIncome,
        payeeAccountNo_get,
        payeeBank_get,
    } = useSelections()

    // 是否存在finVoucherId
    const hasFinVoucherId = computed(() => {
        return formData.finVoucherId
    })

    // 是否互转流水
    const isTransferFlow = computed(() => {
        return formData.bookkeepingType === 0 // 0-互转流水, 1-红冲流水
    })

    // 是否已经关联合同
    const hasContract = computed(() => {
        return formData.finContractId
    })

    // 是否已生成凭证
    const hasVoucher = computed(() => {
        return [4, 5].includes(formData.voucherStatus) //凭证状态，0-暂存，1-未提交，2-审核中，3-已驳回，4-待记账，5-已记账
    })

    /**
     * @description 获取科目名称和收入类型
     * @returns {Object} selectSubject selectInAmountDetail
     */
    function getAccountSubjectAndInAmountDetail() {
        const selectSubject = getNodeByKeyDeep(
            formData.baseAccountSubjectId,
            getSelections('subjectList'),
            {
                fieldKey: 'baseAccountSubjectId',
            },
        )
        let selectInOrOutAmountDetail = null
        if (modalTitle.value.includes('收入')) {
            // 收入类型
            selectInOrOutAmountDetail =
                getNodeById(formData.inAmountDetail, inAmountDetail.value, {
                    fieldKey: 'id',
                }) || formData.inAmountDetail
        } else if (modalTitle.value.includes('支出')) {
            // 支出类型
            selectInOrOutAmountDetail =
                getNodeById(formData.outAmountDetail, outAmountDetail.value, {
                    fieldKey: 'id',
                }) || formData.outAmountDetail
        }

        return {
            selectSubject,
            selectInOrOutAmountDetail,
        }
    }
    /**
     * @description 获取科目名称和收入类型
     * @returns {Object} selectSubject selectInAmountDetail
     */
    function getAccountSubjectAndOutAmountDetail() {
        const selectSubject = getNodeByKeyDeep(
            formData.baseAccountSubjectId,
            getSelections('subjectList'),
            {
                fieldKey: 'baseAccountSubjectId',
            },
        )
        let selectInOrOutAmountDetail = null
        if (modalTitle.value.includes('收入')) {
            // 支出类型
            selectInOrOutAmountDetail =
                getNodeById(formData.inAmountDetail, inAmountDetail.value, {
                    fieldKey: 'id',
                }) || formData.inAmountDetail
        } else if (modalTitle.value.includes('支出')) {
            // 收入类型
            selectInOrOutAmountDetail =
                getNodeById(formData.outAmountDetail, outAmountDetail.value, {
                    fieldKey: 'id',
                }) || formData.outAmountDetail
        }

        return {
            selectSubject,
            selectInOrOutAmountDetail,
        }
    }
    // 是否是库存现金科目-收入编辑
    const isCashAccountSubject_inAmount_edit = ref(false)
    // 是否是银行存款科目-收入编辑
    const isBankAccountSubject_inAmount_edit = ref(false)
    // 是否是库存现金科目-支出编辑
    const isCashAccountSubject_outAmount_edit = ref(false)
    // 是否是银行存款科目-支出编辑
    const isBankAccountSubject_outAmount_edit = ref(false)
    // 是否银行存款&收入支出类型是银行转账
    const isBankAccountSubjectAndAmountDetail = ref(false)

    /**
     * @description 如果是库存现金以及收入类型是提现返回true
     * @param {*} selectSubject 选中的科目
     * @param {*} selectInOrOutAmountDetail 选中的收入/支出类型
     * @returns {boolean}
     */
    function checkIsbankAccountSubject(selectSubject, selectInOrOutAmountDetail) {
        if (modalTitle.value.includes('收入')) {
            return (
                selectSubject?.subjectCode?.startsWith('101') &&
                (selectInOrOutAmountDetail?.name === '提现' || selectInOrOutAmountDetail === '提现')
            )
        } else if (modalTitle.value.includes('支出')) {
            return (
                selectSubject?.subjectCode?.startsWith('101') &&
                (selectInOrOutAmountDetail?.name === '存款' || selectInOrOutAmountDetail === '存款')
            )
        }
    }
    /**
     * @description 判断显示银行存款科目
     * @returns {boolean}
     */
    function checkShowbankAccountSubject() {
        if (formData.baseAccountSubjectId) {
            const { selectSubject, selectInOrOutAmountDetail } =
                getAccountSubjectAndInAmountDetail()
            if (
                checkIsbankAccountSubject(selectSubject, selectInOrOutAmountDetail) ||
                checkIsbankAccountSubjectAndAmountDetail(selectSubject, selectInOrOutAmountDetail)
            ) {
                return true
            } else {
                return false
            }
        } else {
            return false
        }
    }
    /**
     * @description 如果是银行存款以及收入类型是存款返回true
     * @param {*} selectSubject 选中的科目
     * @param {*} selectInOrOutAmountDetail 选中的收入/支出类型
     * @returns {boolean}
     */
    function checkIsCashAccountSubject(selectSubject, selectInOrOutAmountDetail) {
        if (modalTitle.value.includes('收入')) {
            return (
                selectSubject?.subjectCode?.startsWith('102') &&
                (selectInOrOutAmountDetail?.name === '存款' || selectInOrOutAmountDetail === '存款')
            )
        } else if (modalTitle.value.includes('支出')) {
            return (
                selectSubject?.subjectCode?.startsWith('102') &&
                (selectInOrOutAmountDetail?.name === '提现' || selectInOrOutAmountDetail === '提现')
            )
        }
    }

    /**
     * @description 如果是银行存款以及收入/支出类型是银行转账返回true
     * @param {*} selectSubject 选中的科目
     * @param {*} selectInOrOutAmountDetail 选中的收入/支出类型
     * @returns {boolean}
     */
    function checkIsbankAccountSubjectAndAmountDetail(selectSubject, selectInOrOutAmountDetail) {
        return (
            selectSubject?.subjectCode?.startsWith('102') &&
            (selectInOrOutAmountDetail?.name === '银行转账' ||
                selectInOrOutAmountDetail === '银行转账')
        )
    }

    /**
     * @description 判断显示库存现金科目
     * @returns {boolean}
     */
    function checkShowCashAccountSubject() {
        if (formData.baseAccountSubjectId) {
            const { selectSubject, selectInOrOutAmountDetail } =
                getAccountSubjectAndInAmountDetail()
            if (checkIsCashAccountSubject(selectSubject, selectInOrOutAmountDetail)) {
                return true
            } else {
                return false
            }
        } else {
            return false
        }
    }
    // billCount是否用户已经手动输入
    const isUserInputBillCount = ref(false)
    // 是否用户输入摘要
    const isUserInputsummary = ref(false)

    // 表单字段
    const column = {
        title: computed(() => (modalTitle.value.includes('收入') ? '收入信息' : '支出信息')),
        style: {
            maxHeight: '75vh',
            // maxHeight: '600px',
            // width: '600px',
            overflowY: 'auto',
            overflowX: 'hidden',
        },
        fieldsetStyle: {
            gridArea: 'a1',
        },
        column: [
            {
                el: 'tree-select',
                label: '科目名称',
                name: 'baseAccountSubjectId',
                props: {
                    treeData: computed(() => {
                        const subjectList = getSelections('subjectList')
                        let cashItem = null
                        let bankItem = null
                        // 如果是编辑银行存款科目的收入，只能选择银行存款科目
                        if (
                            isCashAccountSubject_inAmount_edit.value ||
                            isCashAccountSubject_outAmount_edit.value
                        ) {
                            cashItem = subjectList.find(item => item.subjectCode === '101')
                            // forEachDeep(subjectList, node => {
                            //     node.disabled = !node?.subjectCode?.startsWith('101')
                            // })
                            if (cashItem) {
                                return [cashItem]
                            }
                        } else if (
                            isBankAccountSubject_inAmount_edit.value ||
                            isBankAccountSubject_outAmount_edit.value ||
                            isBankAccountSubjectAndAmountDetail.value
                        ) {
                            bankItem = subjectList.find(item => item.subjectCode === '102')
                            if (bankItem) {
                                return [bankItem]
                            }
                        }

                        return subjectList
                    }),
                    fieldNames: {
                        children: 'children',
                        label: 'subjectName',
                        value: 'baseAccountSubjectId',
                    },
                    placeholder: '请选择科目名称',
                    disabled: hasFinVoucherId,
                    style: {
                        maxWidth: '370px',
                    },
                },
                event: {
                    onChange: val => {
                        const { selectSubject, selectInOrOutAmountDetail } =
                            getAccountSubjectAndInAmountDetail()
                        if (
                            formData.originSubjectId &&
                            !checkIsCashAccountSubject(selectSubject, selectInOrOutAmountDetail) &&
                            !checkShowbankAccountSubject()
                        ) {
                            formData.originSubjectId = null
                        }
                        if (
                            formData.originSubjectId &&
                            !checkIsbankAccountSubject(selectSubject, selectInOrOutAmountDetail) &&
                            !checkShowCashAccountSubject()
                        ) {
                            formData.originSubjectId = null
                        }
                        if (
                            formData.originSubjectId &&
                            !checkIsbankAccountSubjectAndAmountDetail(
                                selectSubject,
                                selectInOrOutAmountDetail,
                            )
                        ) {
                            formData.originSubjectId = null
                        }
                    },
                },
                rules: [{ required: true, message: '请选择科目名称' }],
            },
            {
                el: 'input',
                label: '流水号',
                name: 'serialNo',
                colspan: 24,
                props: {
                    placeholder: '系统生成',
                    disabled: true,
                },
            },
            {
                el: 'date-picker',
                label: '记账日期',
                name: 'bookkeepingDate',
                colspan: 24,
                props: {
                    valueFormat: 'YYYY-MM-DD',
                    disabledBeforeAccountStartDate: true,
                    disabledDate: time => {
                        // 资产账记账日期不能跨月修改
                        if (currentRecord.value?.source === 1) {
                            // 获取记账月份的第一天和最后一天
                            const startOfMonth = dayjs(
                                currentRecord.value?.bookkeepingDate,
                            ).startOf('month')
                            const endOfMonth = dayjs(currentRecord.value?.bookkeepingDate).endOf(
                                'month',
                            )
                            return time < startOfMonth || time > endOfMonth
                        } else {
                            return (
                                time <
                                    dayjs(
                                        `${currentAccount.value?.currentYear}-${currentAccount.value?.currenMonth}-01`,
                                    ) ||
                                // 不能选今天和今天之后的日期
                                time > dayjs().endOf('day')
                            )
                        }
                    },
                    disabled: hasFinVoucherId,
                },
                rules: [{ required: true, message: '请选择记账日期' }],
            },
            {
                el: 'input-number',
                label: '收入',
                name: 'inAmount',
                colspan: 24,
                props: {
                    placeholder: '请输入收入',
                    addonAfter: '元',
                    formatPrice: true,
                    controls: false,
                    precision: 2,
                    // min: 0.01,
                    // step: 0.01,
                    disabled: computed(() => {
                        // 资产账不能修改流水金额
                        return currentRecord.value?.source === 1 || hasFinVoucherId.value
                    }),
                },
                rules: [{ required: true, message: '请输入收入' }],
                display: computed(() => modalTitle.value?.includes('收入')),
            },
            {
                el: 'input-number',
                label: '支出',
                name: 'outAmount',
                colspan: 24,
                props: {
                    placeholder: '请输入支出',
                    addonAfter: '元',
                    formatPrice: true,
                    controls: false,
                    precision: 2,
                    min: 0.01,
                    step: 0.01,
                    disabled: computed(() => {
                        // 资产账不能修改流水金额
                        return currentRecord.value?.source === 1 || hasFinVoucherId.value
                    }),
                },
                rules: [{ required: true, message: '请输入支出' }],
                display: computed(() => modalTitle.value?.includes('支出')),
            },
            {
                el: 'select',
                name: 'inAmountDetail',
                label: '收入类型',
                options: inAmountDetail,
                props: {
                    mode: 'single',
                    filterOption: filterOption,
                    placeholder: '请输入/选择收入类型',
                    disabled: computed(
                        () =>
                            // isCashAccountSubject_inAmount_edit.value ||
                            // isBankAccountSubject_inAmount_edit.value ||
                            // isBankAccountSubjectAndAmountDetail.value ||
                            isTransferFlow.value,
                    ),
                    style: {
                        maxWidth: '370px',
                    },
                },
                event: {
                    onChange: val => {
                        const { selectSubject, selectInOrOutAmountDetail } =
                            getAccountSubjectAndInAmountDetail()
                        if (
                            formData.originSubjectId &&
                            !checkIsbankAccountSubjectAndAmountDetail(
                                selectSubject,
                                selectInOrOutAmountDetail,
                            )
                        ) {
                            formData.originSubjectId = null
                        }
                        hanldeDeleteContract({
                            selectInAmountDetail: val,
                        })
                    },
                    onDelete: item => {
                        deleteCustomItem(item, 'common', hanldeIncome)
                    },
                },
                rules: [
                    { required: true, message: '请输入/选择收入类型' },
                    {
                        //不能超过50个字，不能有空格
                        pattern: /^[^\s]{1,20}$/,
                        message: '不能超过20个字，不能有空格',
                    },
                ],
                display: computed(() => modalTitle.value?.includes('收入')),
            },
            {
                el: 'checkbox',
                label: ' ',
                name: 'inAmountSaveFlag',
                colspan: 24,
                afterText: '保存常用（下次显示在下拉框里）',
                selfProps: {
                    colon: false,
                    style: {
                        display: 'flex',
                        alignItems: 'center',
                    },
                    extra: '不能超过20个字，不能有空格',
                },
                props: {
                    disabled: computed(() => {
                        return (
                            !formData.inAmountDetail ||
                            inAmountDetail.value.some(
                                item =>
                                    item.label === formData.inAmountDetail ||
                                    item.value === formData.inAmountDetail,
                            )
                        )
                    }),
                },
                display: computed(() => modalTitle.value?.includes('收入')),
            },
            {
                el: 'select',
                name: 'outAmountDetail',
                label: '支出类型',
                options: outAmountDetail,
                props: {
                    mode: 'single',
                    filterOption: filterOption,
                    placeholder: '请输入/选择支出类型',
                    disabled: computed(
                        () =>
                            // isCashAccountSubject_outAmount_edit.value ||
                            // isBankAccountSubject_outAmount_edit.value ||
                            // isBankAccountSubjectAndAmountDetail.value ||
                            isTransferFlow.value,
                    ),
                    style: {
                        maxWidth: '370px',
                    },
                },
                event: {
                    onChange: val => {
                        const { selectSubject, selectInOrOutAmountDetail } =
                            getAccountSubjectAndInAmountDetail()
                        if (
                            formData.originSubjectId &&
                            !checkIsbankAccountSubjectAndAmountDetail(
                                selectSubject,
                                selectInOrOutAmountDetail,
                            )
                        ) {
                            formData.originSubjectId = null
                        }
                        hanldeDeleteContract({
                            selectOutAmountDetail: val,
                        })
                    },
                    onDelete: item => {
                        deleteCustomItem(item, 'common', handleOutcome)
                    },
                },
                rules: [
                    { required: true, message: '请输入/选择支出类型' },
                    {
                        //不能超过50个字，不能有空格
                        pattern: /^[^\s]{1,20}$/,
                        message: '不能超过20个字，不能有空格',
                    },
                ],
                display: computed(() => modalTitle.value?.includes('支出')),
            },
            {
                el: 'checkbox',
                label: ' ',
                name: 'outAmountSaveFlag',
                colspan: 24,
                afterText: '保存常用（下次显示在下拉框里）',
                selfProps: {
                    colon: false,
                    style: {
                        display: 'flex',
                        alignItems: 'center',
                    },
                    extra: '不能超过20个字，不能有空格',
                },
                props: {
                    disabled: computed(() => {
                        return (
                            !formData.outAmountDetail ||
                            outAmountDetail.value.some(
                                item =>
                                    item.label === formData.outAmountDetail ||
                                    item.value === formData.outAmountDetail,
                            )
                        )
                    }),
                },
                display: computed(() => modalTitle.value?.includes('支出')),
            },
            {
                el: 'tree-select',
                name: 'originSubjectId',
                label: '银行存款科目',
                props: {
                    treeData: computed(() => {
                        const subjectList = getSelections('subjectList')
                        let bankItem = null
                        if (
                            isCashAccountSubject_inAmount_edit.value ||
                            isCashAccountSubject_outAmount_edit.value ||
                            isBankAccountSubjectAndAmountDetail.value ||
                            checkShowbankAccountSubject()
                        ) {
                            bankItem = subjectList.find(item => item.subjectCode === '102')
                            if (bankItem) {
                                return [bankItem]
                            }
                        }
                        return subjectList
                    }),
                    fieldNames: {
                        children: 'children',
                        label: 'subjectName',
                        value: 'baseAccountSubjectId',
                    },
                    placeholder: '请选择银行存款科目',
                    style: {
                        maxWidth: '370px',
                    },
                    disabled: isTransferFlow,
                },
                rules: [{ required: true, message: '请选择银行存款科目' }],
                display: computed(() => checkShowbankAccountSubject()),
            },
            {
                el: 'tree-select',
                name: 'originSubjectId',
                label: '库存现金科目',
                props: {
                    treeData: computed(() => {
                        const subjectList = getSelections('subjectList')
                        let bankItem = null
                        if (
                            isBankAccountSubject_inAmount_edit.value ||
                            isBankAccountSubject_outAmount_edit.value ||
                            checkShowCashAccountSubject()
                        ) {
                            bankItem = subjectList.find(item => item.subjectCode === '101')
                            if (bankItem) {
                                return [bankItem]
                            }
                        }

                        return subjectList
                    }),
                    fieldNames: {
                        children: 'children',
                        label: 'subjectName',
                        value: 'baseAccountSubjectId',
                    },
                    placeholder: '请选择库存现金科目',
                    style: {
                        maxWidth: '370px',
                    },
                },
                rules: [{ required: true, message: '请选择库存现金科目' }],
                display: computed(() => checkShowCashAccountSubject()),
            },
            {
                el: 'select',
                name: 'originVoucherList',
                label: '关联业务',
                options: originVoucherList,
                colspan: 24,
                event: {
                    onDelete: item => {
                        deleteCustomItem(item, 'common', handleOutcome)
                    },
                    onChange: val => {
                        hanldeDeleteContract({
                            selectOriginVoucherList: val,
                        })
                    },
                },
                props: {
                    mode: 'multiple-enhance',
                    maxTagCount: 10,
                    maxTagTextLength: 20,
                    filterOption: filterOption,
                    style: {
                        maxWidth: '370px',
                    },
                    disabled: computed(() => {
                        return hasContract.value || hasVoucher.value
                    }),
                },
                rules: [{ required: true, message: '请输入/选择' }],
            },
            {
                el: 'checkbox',
                label: ' ',
                name: 'originVoucherSaveFlag',
                colspan: 24,
                afterText: '保存常用（下次显示在下拉框里）',
                selfProps: {
                    colon: false,
                    style: {
                        display: 'flex',
                        alignItems: 'center',
                    },
                    extra: '支持多个，鼠标离开输入框（失焦）或按Enter键完成一个输入，一个不能超过20个字，不能有空格',
                },
                props: {
                    disabled: computed(() => {
                        return formData.originVoucherList.every(item =>
                            originVoucherList.value.some(v => v.label === item || v.value === item),
                        )
                    }),
                },
            },
            {
                name: 'finContractId',
                formSlot: 'finContractStagingIdList',
                label: '关联合同',
                display: computed(() => {
                    if (formData.finContractStagingIdList?.length > 0) {
                        return true
                    }
                    let list = []
                    // 如果formData.originVoucherList中有合同，显示合同
                    if (formData.originVoucherList) {
                        formData.originVoucherList.forEach(item => {
                            let obj = findTargetIdToLabel(item, originVoucherList.value)
                            if (obj?.includes('合同')) {
                                list.push(obj)
                            }
                        })
                    }
                    return list.length > 0
                }),
                selfProps: {
                    rules: [{ required: true, message: '请选择合同与合同分期' }],
                },
            },
            {
                el: 'select',
                name: 'summary',
                label: '摘要',
                options: abstractList,
                colspan: 24,
                props: {
                    mode: 'single',
                    filterOption,
                    allowClear: true,
                    style: {
                        maxWidth: '370px',
                    },
                },
                event: {
                    onDelete: item => {
                        deleteCustomItem(item, 'abstract', handleOutcome)
                    },
                    onChange: val => {
                        isUserInputsummary.value = true
                    },
                },
                rules: [
                    { required: true, message: '请输入/选择' },
                    {
                        //不能超过50个字，不能有空格
                        pattern: /^[^\s]{1,100}$/,
                        message: '不能超过100个字，不能有空格',
                    },
                ],
            },
            {
                el: 'checkbox',
                label: ' ',
                name: 'summarySaveFlag',
                colspan: 24,
                afterText: '保存常用（下次显示在下拉框里）',
                selfProps: {
                    colon: false,
                    style: {
                        display: 'flex',
                        alignItems: 'center',
                    },
                    extra: '不能超过100个字，不能有空格',
                },
                props: {
                    disabled: computed(() => {
                        return (
                            !formData.summary ||
                            abstractList.value.some(
                                item =>
                                    item.label === formData.summary ||
                                    item.value === formData.summary,
                            )
                        )
                    }),
                },
            },
            {
                el: 'input',
                label: '余额',
                name: 'balance',
                colspan: 24,
                props: {
                    placeholder: '系统生成',
                    addonAfter: '元',
                    formatter: value => formatPriceToFixed(value, 2),
                    parser: value => value.replace(/\$\s?|(,*)/g, ''),
                    controls: false,
                    precision: 2,
                    min: 0,
                    step: 0.01,
                    disabled: true,
                },
            },
            {
                name: 'billUrlList',
                formSlot: 'billUrlList',
                label: computed(() => {
                    let subjectName = '银行'
                    let payType = ''
                    // 如果是收入,payType是收款,如果是支出,payType是付款
                    if (modalTitle.value.includes('收入')) {
                        payType = '收款'
                    } else if (modalTitle.value.includes('支出')) {
                        payType = '付款'
                    }

                    // 如果科目名称是库存现金，subjectName是现金,否则是银行
                    if (formData.baseAccountSubjectId) {
                        const { selectSubject, selectInOrOutAmountDetail } =
                            getAccountSubjectAndInAmountDetail()
                        if (selectSubject?.subjectCode?.startsWith('101')) {
                            subjectName = '现金'
                        } else if (selectSubject?.subjectCode?.startsWith('102')) {
                            subjectName = '银行'
                        }
                    }
                    return `${subjectName}${payType}原始凭证`
                }),
                selfProps: {
                    rules: [{ required: true, message: '请上传原始凭证' }],
                },
            },
            {
                name: 'otherBillUrlList',
                formSlot: 'otherBillUrlList',
                label: '其它原始凭证',
                // selfProps: {
                //     rules: [{ required: true, message: '请上传原始凭证' }],
                // },
            },
            {
                el: 'input-number',
                label: '单据数量',
                name: 'billCount',
                colspan: 24,
                props: {
                    controls: false,
                    min: 1,
                    precision: 0,
                    disabled: hasVoucher,
                },
                rules: [{ required: true, message: '请输入单据数量' }],
                event: {
                    onChange: val => {
                        isUserInputBillCount.value = true
                    },
                },
            },
            {
                el: 'select',
                label: '结算方式',
                name: 'settlementMode',
                colspan: 24,
                options: settlementMode,
                props: {
                    filterOption,
                },
            },
            {
                el: 'input',
                label: '结算号',
                name: 'settlementNo',
                colspan: 24,
                props: {
                    controls: false,
                },
            },
            {
                el: 'input',
                label: '用途',
                name: 'purpose',
                colspan: 24,
                display: computed(() => modalTitle.value?.includes('支出')),
            },
        ],
    }
    const column2 = {
        title: '干系人信息',
        style: {
            maxHeight: '75vh',
            // maxHeight: '600px',
            // width: '600px',
            paddingLeft: '24px',
            overflowY: 'auto',
            overflowX: 'hidden',
        },
        fieldsetStyle: {
            gridArea: 'a2',
        },
        column: [
            // 本单位经手人
            {
                el: 'select',
                name: 'handlerList',
                label: '本单位经手人',
                colspan: 24,
                options: handlerList,
                event: {
                    onDelete: item => {
                        deleteCustomItem(item, 'stakeholder', hanldeIncome)
                    },
                },
                props: {
                    mode: 'multiple-enhance',
                    maxTagCount: 10,
                    maxTagTextLength: 20,
                    filterOption,
                    style: {
                        maxWidth: '370px',
                    },
                },
                rules: [{ required: true, message: '请输入/选择' }],
            },
            {
                el: 'checkbox',
                label: ' ',
                name: 'handlerSaveFlag',
                colspan: 24,
                afterText: '保存常用（下次显示在下拉框里）',
                selfProps: {
                    colon: false,
                    style: {
                        display: 'flex',
                        alignItems: 'center',
                    },
                    extra: '支持多个，鼠标离开输入框（失焦）或按Enter键完成一个输入，一个不能超过20个字，不能有空格',
                },
                props: {
                    disabled: computed(() => {
                        return formData.handlerList.every(item =>
                            handlerList.value.some(v => v.value === item || v.label === item),
                        )
                    }),
                },
            },
            // 审批人
            {
                el: 'select',
                name: 'auditorList',
                label: '审批人',
                options: auditorList,
                event: {
                    onDelete: item => {
                        deleteCustomItem(item, 'stakeholder', handleOutcome)
                    },
                },
                props: {
                    mode: 'multiple-enhance',
                    maxTagCount: 10,
                    maxTagTextLength: 20,
                    filterOption,
                    style: {
                        maxWidth: '370px',
                    },
                },
                rules: [{ required: true, message: '请输入/选择审批人' }],
                display: computed(() => modalTitle.value?.includes('支出')),
            },
            {
                el: 'checkbox',
                label: ' ',
                name: 'auditorSaveFlag',
                colspan: 24,
                afterText: '保存常用（下次显示在下拉框里）',
                selfProps: {
                    colon: false,
                    style: {
                        display: 'flex',
                        alignItems: 'center',
                    },
                    extra: '支持多个，鼠标离开输入框（失焦）或按Enter键完成一个输入，一个不能超过20个字，不能有空格',
                },
                props: {
                    disabled: computed(() => {
                        return formData.auditorList.every(item =>
                            auditorList.value.some(v => v.value === item || v.label === item),
                        )
                    }),
                },
                display: computed(() => modalTitle.value?.includes('支出')),
            },
            {
                el: 'select',
                name: 'payeeList',
                label: '收款单位/人',
                // label: '收款人',
                options: payeeList,
                props: {
                    mode: 'multiple-enhance',
                    maxTagCount: 10,
                    maxTagTextLength: 64,
                    filterOption,
                    style: {
                        maxWidth: '370px',
                    },
                },
                event: {
                    onDelete: item => {
                        deleteCustomItem(item, 'stakeholder', hanldeIncome)
                    },
                },
                rules: [
                    {
                        required: computed(() =>
                            modalTitle.value?.includes('收入') ? false : true,
                        ),
                        message: '请输入/选择',
                    },
                ],
                display: computed(() => modalTitle.value?.includes('支出')),
            },
            {
                el: 'checkbox',
                label: ' ',
                name: 'payeeSaveFlag',
                colspan: 24,
                afterText: '保存常用（下次显示在下拉框里）',
                selfProps: {
                    colon: false,
                    style: {
                        display: 'flex',
                        alignItems: 'center',
                    },
                    extra: '支持多个，鼠标离开输入框（失焦）或按Enter键完成一个输入，一个不能超过64个字，不能有空格',
                },
                props: {
                    disabled: computed(() => {
                        return formData.payeeList.every(item =>
                            payeeList.value.some(v => v.value === item || v.label === item),
                        )
                    }),
                },
                display: computed(() => modalTitle.value?.includes('支出')),
            },
            // 付款方单位/人 - 收入
            {
                el: 'select',
                name: 'billerList',
                label: '付款方单位/人',
                options: billerList,
                event: {
                    onDelete: item => {
                        deleteCustomItem(item, 'stakeholder', handleOutcome)
                    },
                },
                props: {
                    mode: 'multiple-enhance',
                    maxTagCount: 10,
                    maxTagTextLength: 64,
                    filterOption,
                    style: {
                        maxWidth: '370px',
                    },
                },
                rules: [{ required: true, message: '请输入/选择付款方单位/人' }],
                display: computed(() => modalTitle.value?.includes('收入')),
            },
            {
                el: 'checkbox',
                label: ' ',
                name: 'billerSaveFlag',
                colspan: 24,
                afterText: '保存常用（下次显示在下拉框里）',
                selfProps: {
                    colon: false,
                    style: {
                        display: 'flex',
                        alignItems: 'center',
                    },
                    extra: '支持多个，鼠标离开输入框（失焦）或按Enter键完成一个输入，一个不能超过64个字，不能有空格',
                },
                props: {
                    disabled: computed(() => {
                        return formData.billerList.every(item =>
                            billerList.value.some(v => v.value === item || v.label === item),
                        )
                    }),
                },
                display: computed(() => modalTitle.value?.includes('收入')),
            },
            // 收入：付款方 / 支出：收款 账户
            {
                el: 'select',
                name: 'payeeAccountType',
                label: computed(() =>
                    modalTitle.value?.includes('收入') ? '付款方账号' : '收款账号',
                ),
                props: {
                    style: {
                        maxWidth: '370px',
                    },
                },
                options: [
                    {
                        label: '银行卡号',
                        value: 1,
                    },
                    {
                        label: '非银行卡号',
                        value: 2,
                    },
                ],
                event: {
                    onChange: val => {
                        if (val === 2) {
                            formData.payeeAccountNo = '现金账号'
                        } else {
                            formData.payeeAccountNo = null
                        }
                    },
                },
                rules: [{ required: true, message: '请选择账户类型' }],
            },
            {
                el: 'select',
                name: 'payeeAccountNo',
                label: ' ',
                options: computed(() => {
                    // 如果是收入
                    if (modalTitle.value.includes('收入')) {
                        return payeeAccountNo.value
                    } else {
                        return payeeAccountNo_get.value
                    }
                }),
                selfProps: {
                    colon: false,
                    style: {
                        display: 'flex',
                        alignItems: 'center',
                    },
                },
                props: {
                    mode: 'single',
                    filterOption: filterOption,
                    placeholder: '请输入/选择银行卡号',
                    disabled: computed(
                        () => formData.payeeAccountType === 2 || formData.payeeAccountType === null,
                    ),
                    style: {
                        maxWidth: '370px',
                    },
                },
                event: {
                    onDelete: item => {
                        const _item = {
                            ...item,
                            value: item.commonDataItemId,
                        }
                        deleteCustomItem(_item, 'delCommonDataItem', () => {
                            hanldeIncome()
                            handleOutcome()
                        })
                    },
                },
                hideRequiredMark: true,
                rules: [
                    { required: true, message: '请输入或选择银行卡号' },
                    {
                        validator: (rule, value) => {
                            if (formData.payeeAccountType === 1) {
                                if (!value) {
                                    return Promise.reject('请输入或选择银行卡号')
                                } else if (!/^\d{1,64}$/.test(value)) {
                                    return Promise.reject('长度不能超过64个数字，不能有空格')
                                } else {
                                    return Promise.resolve()
                                }
                            } else {
                                return Promise.resolve()
                            }
                        },
                        message: '长度不能超过64个数字，不能有空格',
                    },
                ],
            },
            {
                el: 'checkbox',
                label: ' ',
                name: 'payeeAccountNoSaveFlag',
                colspan: 24,
                afterText: '保存常用（下次显示在下拉框里）',
                selfProps: {
                    colon: false,
                    style: {
                        display: 'flex',
                        alignItems: 'center',
                    },
                    extra: '长度不能超过64个数字，不能有空格',
                },
                props: {
                    disabled: computed(() => {
                        return (
                            !formData.payeeAccountNo ||
                            formData.payeeAccountType === 2 ||
                            payeeAccountNo.value.some(
                                item =>
                                    item.label === formData.payeeAccountNo ||
                                    item.value === formData.payeeAccountNo,
                            )
                        )
                    }),
                },
            },
            {
                el: 'select',
                name: 'payeeBank',
                label: computed(() =>
                    modalTitle.value?.includes('收入') ? '付款方银行名称' : '收款银行名称',
                ),
                options: computed(() => {
                    if (modalTitle.value.includes('收入')) {
                        return payeeBank.value
                    }
                    return payeeBank_get.value
                }),
                selfProps: {
                    colon: false,
                    style: {
                        display: 'flex',
                        alignItems: 'center',
                    },
                },
                props: {
                    mode: 'single',
                    filterOption: filterOption,
                    placeholder: `请输入/选择${modalTitle.value?.includes('收入') ? '付款方' : '收款'}银行名称`,
                    style: {
                        maxWidth: '370px',
                    },
                },
                event: {
                    onDelete: item => {
                        const _item = {
                            ...item,
                            value: item.commonDataItemId,
                        }
                        deleteCustomItem(_item, 'delCommonDataItem', () => {
                            hanldeIncome()
                            handleOutcome()
                        })
                    },
                },
                rules: [
                    // { required: true, message: '请输入银行名称' },
                    {
                        pattern: /^[^\s]{1,64}$/,
                        message: '长度不能超过64个字，支持数字、字母，不能有空格',
                    },
                ],
            },
            {
                el: 'checkbox',
                label: ' ',
                name: 'payeeBankSaveFlag',
                colspan: 24,
                afterText: '保存常用（下次显示在下拉框里）',
                selfProps: {
                    colon: false,
                    style: {
                        display: 'flex',
                        alignItems: 'center',
                    },
                    extra: '长度不能超过64个字，支持数字、字母，不能有空格',
                },
                props: {
                    disabled: computed(() => {
                        return (
                            !formData.payeeBank ||
                            payeeBank.value.some(
                                item =>
                                    item.label === formData.payeeBank ||
                                    item.value === formData.payeeBank,
                            )
                        )
                    }),
                },
            },
            // 证明人
            {
                el: 'select',
                name: 'certifierList',
                label: '证明人',
                options: certifierList,
                props: {
                    mode: 'multiple-enhance',
                    maxTagCount: 10,
                    maxTagTextLength: 20,
                    filterOption,
                    style: {
                        maxWidth: '370px',
                    },
                },
                event: {
                    onDelete: item => {
                        deleteCustomItem(item, 'stakeholder', handleOutcome)
                    },
                },
                display: computed(() => modalTitle.value?.includes('支出')),
            },
            {
                el: 'checkbox',
                label: ' ',
                name: 'certifierSaveFlag',
                colspan: 24,
                afterText: '保存常用（下次显示在下拉框里）',
                selfProps: {
                    colon: false,
                    style: {
                        display: 'flex',
                        alignItems: 'center',
                    },
                    extra: '支持多个，鼠标离开输入框（失焦）或按Enter键完成一个输入，一个不能超过20个字，不能有空格',
                },
                props: {
                    disabled: computed(() => {
                        return formData.certifierList.every(item =>
                            certifierList.value.some(v => v.value === item || v.label === item),
                        )
                    }),
                },
                display: computed(() => modalTitle.value?.includes('支出')),
            },
            // 验收人
            {
                el: 'select',
                name: 'accepterList',
                label: '验收人',
                options: accepterList,
                props: {
                    mode: 'multiple-enhance',
                    maxTagCount: 10,
                    maxTagTextLength: 20,
                    filterOption,
                    style: {
                        maxWidth: '370px',
                    },
                },
                event: {
                    onDelete: item => {
                        deleteCustomItem(item, 'stakeholder', handleOutcome)
                    },
                },
                display: computed(() => modalTitle.value?.includes('支出')),
            },
            {
                el: 'checkbox',
                label: ' ',
                name: 'accepterSaveFlag',
                colspan: 24,
                afterText: '保存常用（下次显示在下拉框里）',
                selfProps: {
                    colon: false,
                    style: {
                        display: 'flex',
                        alignItems: 'center',
                    },
                    extra: '支持多个，鼠标离开输入框（失焦）或按Enter键完成一个输入，一个不能超过20个字，不能有空格',
                },
                props: {
                    disabled: computed(() => {
                        return formData.accepterList.every(item =>
                            accepterList.value.some(v => v.value === item || v.label === item),
                        )
                    }),
                },
                display: computed(() => modalTitle.value?.includes('支出')),
            },
        ],
    }
    const formModel = ref([column, column2])
    let summaryWatch = null
    // 自动生成摘要
    function watchForm() {
        if (modalTitle.value.includes('收入')) {
            summaryWatch = watch(
                () => [
                    formData.baseAccountSubjectId,
                    formData.bookkeepingDate,
                    formData.inAmount,
                    formData.inAmountDetail,
                ],
                ([baseAccountSubjectId, bookkeepingDate, inAmount, inAmountDetail]) => {
                    if (
                        baseAccountSubjectId &&
                        bookkeepingDate &&
                        inAmount &&
                        inAmountDetail &&
                        !isUserInputsummary.value
                    ) {
                        const selectSubject = getNodeByKeyDeep(
                            baseAccountSubjectId,
                            getSelections('subjectList'),
                            {
                                fieldKey: 'baseAccountSubjectId',
                            },
                        )
                        const selectInAmountDetail = getNodeByKeyDeep(
                            inAmountDetail,
                            getSelections('inAmountDetail'),
                            {
                                fieldKey: 'id',
                            },
                        )
                        const selectInAmountDetailName =
                            selectInAmountDetail?.name || inAmountDetail
                        const subjectName = selectSubject?.subjectName.replace(/\s+/g, '')
                        const _summary = `${subjectName}${dayjs(bookkeepingDate).format(
                            'YYYY年MM月DD日',
                        )}${selectInAmountDetailName}${inAmount}元`
                        editForm({ summary: _summary })
                    }
                },
            )
        } else if (modalTitle.value.includes('支出')) {
            summaryWatch = watch(
                () => [
                    formData.baseAccountSubjectId,
                    formData.bookkeepingDate,
                    formData.outAmount,
                    formData.outAmountDetail,
                ],
                ([baseAccountSubjectId, bookkeepingDate, outAmount, outAmountDetail]) => {
                    if (
                        baseAccountSubjectId &&
                        bookkeepingDate &&
                        outAmount &&
                        outAmountDetail &&
                        !isUserInputsummary.value
                    ) {
                        const selectSubject = getNodeByKeyDeep(
                            baseAccountSubjectId,
                            getSelections('subjectList'),
                            {
                                fieldKey: 'baseAccountSubjectId',
                            },
                        )
                        const selectOutAmountDetail = getNodeByKeyDeep(
                            outAmountDetail,
                            getSelections('outAmountDetail'),
                            {
                                fieldKey: 'id',
                            },
                        )
                        const selectOutAmountDetailName =
                            selectOutAmountDetail?.name || outAmountDetail
                        const subjectName = selectSubject?.subjectName.replace(/\s+/g, '')
                        const _summary = `${subjectName}${dayjs(bookkeepingDate).format(
                            'YYYY年MM月DD日',
                        )}${selectOutAmountDetailName}${outAmount}元`
                        editForm({ summary: _summary })
                    }
                },
            )
        }
    }

    // 关联业务删除了合同时，清空关联合同
    watch(
        () => formData.originVoucherList,
        (newVal, oldVal) => {
            const diff = oldVal.filter(item => !newVal.includes(item))
            const label = findTargetIdToLabel(diff[0], originVoucherList.value)
            // 如果删了合同，且之前关联合同有值，则删除关联合同
            if (label === '合同' && formData.finContractId) {
                // 清空关联合同
                formData.finContractId = null
                formData.finContractStagingIdList = null
                SelectButtonRef.value?.resetForm()
                SelectButtonRef.value?.handleDelete()
            }
        },
    )

    /**
     * @description: 处理关联业务中的合同项
     */
    function hanldeDeleteContract({
        selectOriginVoucherList = null, // 选择的关联业务
        selectInAmountDetail = null, // 选择的收入类型
        selectOutAmountDetail = null, // 选择的支出类型
    } = {}) {
        let select
        if (currentAmountType.value === 0) {
            // 获取选中的收入类型
            select = findTargetIdToLabel(formData.inAmountDetail, inAmountDetail.value)
        } else {
            // 获取选中的支出类型
            select = findTargetIdToLabel(formData.outAmountDetail, outAmountDetail.value)
        }
        if (selectInAmountDetail || selectOutAmountDetail) {
            if (select) {
                if (formData.originVoucherList?.length > 0) {
                    formData.originVoucherList.some(i => {
                        const target = findTargetIdToLabel(i, originVoucherList.value)
                        if (target === '合同') {
                            if (select === '提现' || select === '存款' || select === '银行转账') {
                                if (currentAmountType.value === 0) {
                                    // 清空收入类型
                                    formData.inAmountDetail = null
                                } else {
                                    // 清空支出类型
                                    formData.outAmountDetail = null
                                }
                                message.warning(
                                    `选择的${currentAmountType.value === 0 ? '收入' : '支出'}类型会产生出纳互转流水，会影响合同关联，请重新选择。`,
                                )
                            }
                        }
                    })
                }
            }
        } else if (selectOriginVoucherList) {
            if (formData.originVoucherList?.length > 0) {
                formData.originVoucherList.some(i => {
                    const target = findTargetIdToLabel(i, originVoucherList.value)
                    if (target === '合同') {
                        if (select === '提现' || select === '存款' || select === '银行转账') {
                            // 删除合同项
                            formData.originVoucherList = formData.originVoucherList.filter(id => {
                                const label = findTargetIdToLabel(id, originVoucherList.value)
                                return label !== '合同'
                            })
                            // 清空关联合同
                            formData.finContractStagingIdList = null
                            SelectButtonRef.value?.resetForm()
                            SelectButtonRef.value?.handleDelete()
                            message.warning(
                                `选择的${currentAmountType.value === 0 ? '收入' : '支出'}类型会产生出纳互转流水，会影响合同关联，请重新选择。`,
                            )
                        }
                    }
                })
            }
        }
    }

    /**
     * 提交
     * @param {boolean} submit 是否提交 false-暂存 true-保存
     * @returns
     */
    function submit(isSubmit, params = {}) {
        return new Promise(async (resolve, reject) => {
            loading.value = true
            await sleep(500)
            let api = null
            let _params = {}
            const selectSubject = getNodeByKeyDeep(
                formData.baseAccountSubjectId,
                getSelections('subjectList'),
                {
                    fieldKey: 'baseAccountSubjectId',
                },
            )
            _params = {
                ...formData,
                //  temporarySave(是否暂存 0=否 1=是)
                temporarySave: !isSubmit,
                summary: transformParams(formData.summary, 'abstractList', 'string'), // 摘要
                originVoucherList: transformParams(formData.originVoucherList, 'originVoucher'), // 原始凭证来源
                handlerList: transformParams(formData.handlerList, 'handlerList'), //  本单位经手人
                billerList: transformParams(formData.billerList, 'billerList'), // 付款方单位/人
                payeeList: transformParams(formData.payeeList, 'payeeList'), //    收款人 / 收款单位/人
                baseAccountSubjectName: selectSubject?.subjectName,
                billUrlList: formData.billUrlList?.filter(i => i.fileUrl),
                otherBillUrlList: formData.otherBillUrlList?.filter(i => i.fileUrl),
                // payeeAccountNo: transformParams(
                //     formData.payeeAccountNo,
                //     'payeeAccountNo',
                //     'string',
                // ), // 收款账户
                // payeeBank: transformParams(formData.payeeBank, 'payeeBank', 'string'), // 收款银行名称
                // originSubjectId: formData.bankAccountSubjectId || formData.cashAccountSubjectId,
                ...params,
                finContractStagingIdList: formData?.finContractStagingIdList?.filter(i => !!i),
            }
            // delete _params.bankAccountSubjectId
            // delete _params.cashAccountSubjectId
            if (modalTitle.value.includes('收入')) {
                api = apis.bookkeeping.addIncome
                // 如果是暂存
                if (!isSubmit) {
                    api = apis.bookkeeping.addIncomeTemporary
                }
                // 如果是编辑
                if (_params.finBookkeepingId) {
                    api = apis.bookkeeping.addIncomeEdit
                    // 如果是暂存
                    if (!isSubmit) {
                        api = apis.bookkeeping.addIncomeEditTemporary
                    }
                }
                _params = {
                    ..._params,
                    inAmountDetail: transformParams(
                        formData.inAmountDetail,
                        'inAmountDetail',
                        'string',
                    ), // 收入类型
                    payeeList: transformParams(formData.payeeList, 'payeeList'), //    收款人
                    auditorList: transformParams(formData.auditorList, 'auditorList'), //  审批人
                }
            } else if (modalTitle.value.includes('支出')) {
                api = apis.bookkeeping.addOutcome
                // 如果是暂存
                if (!isSubmit) {
                    api = apis.bookkeeping.addOutcomeTemporary
                }
                // 如果是编辑
                if (_params.finBookkeepingId) {
                    api = apis.bookkeeping.addOutcomeEdit
                    // 如果是暂存
                    if (!isSubmit) {
                        api = apis.bookkeeping.addOutcomeEditTemporary
                    }
                }
                _params = {
                    ..._params,
                    outAmountDetail: transformParams(
                        formData.outAmountDetail,
                        'outAmountDetail',
                        'string',
                    ), // 支出类型
                    auditorList: transformParams(formData.auditorList, 'auditorList'), //  审批人
                    certifierList: transformParams(formData.certifierList, 'certifierList'), //    证明人
                    accepterList: transformParams(formData.accepterList, 'accepterList'), //   验收人
                }
            }
            try {
                let { code, message: _message, data } = await api(_params)
                if (code === 8101) {
                    Modal.confirm({
                        title: `提示`,
                        content: _message,
                        okText: '确定',
                        cancelText: '取消',
                        async onOk() {
                            await submit(isSubmit, {
                                confirm: true, // 科目名称变更确认标识 true-同意;false-不同意
                            })
                        },
                        async onCancel() {
                            await submit(isSubmit, {
                                confirm: false,
                            })
                        },
                    })
                } else if (code === 9007) {
                    message.warn(_message)
                    successCallback && (await successCallback())
                    showModal.value = false
                } else if (code === 8606) {
                    // 合同分期问题
                    if (formData.finContractStagingIdList) {
                        // formData.finContractStagingIdList,
                        SelectButtonRef && SelectButtonRef.value?.reRreshStagingList()
                    }
                    message.warn(data)
                } else {
                    successCallback && (await successCallback())
                    message.success(_message)
                    showModal.value = false
                }

                resolve(true)
            } catch (error) {
                reject(error)
            } finally {
                loading.value = false
            }
        })
    }

    function confirmAdd() {
        submit(true)
    }

    // 确认关闭
    async function confirmClose() {
        if (
            (formData.temporarySave === true || formData.temporarySave === null) &&
            Object.keys(compareObjects(oldFormData.value, formData)).length > 0
        ) {
            Modal.confirm({
                title: `数据暂存`,
                cancelText: '否',
                okText: '是',
                content: `是否暂存数据？`,
                async onOk() {
                    await submit(false)
                },
                onCancel() {
                    showModal.value = false
                },
            })
        } else {
            showModal.value = false
        }
    }

    const loading = ref(false)
    const currentRecord = ref({})

    let watchBalandeClear = null
    // 监听科目名称和记账日期，有变动就调取接口获取余额
    function watchBalande() {
        return watch(
            () => [formData.baseAccountSubjectId, formData.bookkeepingDate],
            async ([baseAccountSubjectId, bookkeepingDate]) => {
                if (baseAccountSubjectId && bookkeepingDate) {
                    try {
                        const { data } = await apis.bookkeeping.getTellerBalance4TS({
                            baseAccountSubjectId,
                            bookkeepingDate,
                        })
                        editForm({ balance: data.balance })
                    } catch (error) {
                        message.error(error)
                    }
                }
            },
        )
    }

    let watchBillUrlListClear = null
    function watchBillUrlList() {
        // 自动修改modalFormData.billUrlList
        return watch(
            [() => formData.billUrlList, () => formData.otherBillUrlList],
            ([billUrlList, otherBillUrlList]) => {
                // 只会在用户没有手动输入单据数量的时候才会自动修改
                if (
                    (billUrlList?.length || otherBillUrlList?.length) &&
                    !isUserInputBillCount.value
                ) {
                    formData.billCount =
                        (billUrlList?.length ?? 0) + (otherBillUrlList?.length ?? 0)
                }
            },
        )
    }

    watch(showModal, val => {
        if (!val) {
            resetForm()
            SelectButtonRef.value?.resetForm()
            isUserInputBillCount.value = false
            isUserInputsummary.value = false
            summaryWatch && summaryWatch()
            summaryWatch = null
            watchBillUrlListClear && watchBillUrlListClear()
            watchBillUrlListClear = null
            watchBalandeClear && watchBalandeClear()
            watchBalandeClear = null
            isCashAccountSubject_inAmount_edit.value = false
            isBankAccountSubject_inAmount_edit.value = false
            isCashAccountSubject_outAmount_edit.value = false
            isBankAccountSubject_outAmount_edit.value = false
            isBankAccountSubjectAndAmountDetail.value = false
        }
    })
    const currentAmountType = ref(0) // 0-收入；1-支出
    const currentAddType = ref(null) // 新增类型
    async function handleShowModal(type, record) {
        currentRecord.value = record
        // amountType 收入/支出类型，0-收入；1-支出
        if (type.includes('新增')) {
            currentAddType.value = type
            formData.baseAccountSubjectId = activeMenuItem.value?.baseAccountSubjectId
            watchBalandeClear = watchBalande()
            watchBillUrlListClear = watchBillUrlList()
            nextTick(() => {
                oldFormData.value = _.cloneDeep(formData)
            })
        }
        if (type === '收入新增') {
            modalTitle.value = '收入新增'
            hanldeIncome()
            watchForm()
            // handleBookkeepingInfo('', {
            //     amountType: 0,
            //     subjectCode: activeMenuItem.value?.subjectCode,
            // })
        } else if (type === '支出新增') {
            modalTitle.value = '支出新增'
            handleOutcome()
            watchForm()
            // handleBookkeepingInfo('', {
            //     amountType: 1,
            //     subjectCode: activeMenuItem.value?.subjectCode,
            // })
        } else if (type === '编辑') {
            // amountType 收入/支出类型，0-收入；1-支出
            const { amountType } = record || {}
            currentAddType.value = null
            if (amountType === 0) {
                modalTitle.value = '收入编辑'
            } else if (amountType === 1) {
                modalTitle.value = '支出编辑'
            }
            watchBalandeClear = watchBalande()
            if (amountType === 0) {
                //收入编辑
                await hanldeIncome()
            } else if (amountType === 1) {
                //支出编辑
                await handleOutcome()
            }
            await handleBookkeepingInfo(record.finBookkeepingId)
            watchBillUrlListClear = watchBillUrlList()
        }
        currentAmountType.value = modalTitle.value?.includes('收入') ? 0 : 1
        showModal.value = true
    }

    async function handleBookkeepingInfo(finBookkeepingId = '', otherParams = null) {
        loading.value = true
        return await getBookkeepingInfo(finBookkeepingId, otherParams)
            .then(data => {
                const finContractStagingIdList =
                    data?.contractStagingList?.contractStagingList.map(
                        i => i.finContractStagingId,
                    ) || []
                const _data = {
                    ...data,
                    inAmountDetail: getFinalShowValue(
                        transformJSONStringToObject(data.inAmountDetail),
                    ),
                    outAmountDetail: getFinalShowValue(
                        transformJSONStringToObject(data.outAmountDetail),
                    ),
                    summary: getFinalShowValue(transformJSONStringToObject(data.summary), [
                        'baseAbstractId',
                        'abstractContent',
                    ]),
                    // payeeAccountNo: getFinalShowValue(
                    //     transformJSONStringToObject(data.payeeAccountNo),
                    // ),
                    // payeeBank: getFinalShowValue(transformJSONStringToObject(data.payeeBank)),
                    originVoucherList: transformListToIds(data.originVoucherList),
                    handlerList: transformListToIds(data.handlerList),
                    billerList: transformListToIds(data.billerList),
                    payeeList: transformListToIds(data.payeeList),
                    auditorList: transformListToIds(data.auditorList),
                    certifierList: transformListToIds(data.certifierList),
                    accepterList: transformListToIds(data.accepterList),
                    billCount: data.billCount === null ? data.billUrlList?.length : data.billCount,
                    ...data?.relation,
                    totalNumberOfInstallments: data?.relation?.totalPeriods,
                    // 初次赋值，设置合同分期id
                    finContractStagingIdList: data?.relation?.contractStagingList // 默认是选中的数据
                        ?.map(i => i.finContractStagingId),
                    // bankAccountSubjectId: data.originSubjectId,
                    // cashAccountSubjectId: data.originSubjectId,
                }
                if (!finBookkeepingId) {
                    // 填充默认的记账日期
                    _data.bookkeepingDate = formData.bookkeepingDate
                }
                editForm(_data)
                const _inAmountDetail = transformJSONStringToObject(data.inAmountDetail)
                const _outAmountDetail = transformJSONStringToObject(data.outAmountDetail)
                if (modalTitle.value.includes('收入')) {
                    // 判断是否是现金科目
                    if (checkIsbankAccountSubject(data, _inAmountDetail)) {
                        isCashAccountSubject_inAmount_edit.value = true
                    }
                    // 判断是否是银行存款科目
                    else if (checkIsCashAccountSubject(data, _inAmountDetail)) {
                        isBankAccountSubject_inAmount_edit.value = true
                    }
                    // 判断是否是银行存款科目和支出类型是银行转账
                    if (checkIsbankAccountSubjectAndAmountDetail(data, _inAmountDetail)) {
                        isBankAccountSubjectAndAmountDetail.value = true
                    }
                } else if (modalTitle.value.includes('支出')) {
                    // 判断是否是现金科目
                    if (checkIsbankAccountSubject(data, _outAmountDetail)) {
                        isCashAccountSubject_outAmount_edit.value = true
                    } else if (checkIsCashAccountSubject(data, _outAmountDetail)) {
                        isBankAccountSubject_outAmount_edit.value = true
                    }
                    // 判断是否是银行存款科目和支出类型是银行转账
                    if (checkIsbankAccountSubjectAndAmountDetail(data, _outAmountDetail)) {
                        isBankAccountSubjectAndAmountDetail.value = true
                    }
                }

                // watchForm()
                if (!_data.temporarySave) {
                    // 如果不是暂存的数据，就不需要自动计算余额
                    watchBalandeClear && watchBalandeClear()
                    watchBalandeClear = null
                }
                setTimeout(() => {
                    SelectButtonRef.value?.setCurrentContract({
                        ..._data.relation,
                        defaultFinContractId: _data.relation?.finContractId,
                        // 初次赋值，设置选中的合同分期id，回显勾选
                        defaultContractPeriodList: _data.relation?.contractStagingList?.map(i => {
                            return {
                                ...i,
                                totalNumberOfInstallments: _data.relation.totalPeriods,
                            }
                        }),
                    })
                    oldFormData.value = _.cloneDeep(formData)
                }, 300)

                loading.value = false
            })
            .catch(err => {
                loading.value = false
                message.error(err)
            })
    }

    // 原始凭证上传弹窗
    const voucherUploadModalOpen = ref(false)
    function clickVoucher(record) {
        voucherUploadModalOpen.value = true
        currentRecord.value = record
    }

    // 选择合同
    function selectContract(params) {
        editForm({
            finContractId: params.finContractId, //合同id
            finContractStagingIdList: params.finContractStagingIdList, //合同分期id
            contractName: params.contractName, //合同名称
            contractSerialNumber: params.contractSerialNumber, //合同编号
            periods: params.periods, //合同期数
            totalNumberOfInstallments: params.totalNumberOfInstallments, //总期数
            performanceAmount: params.performanceAmount, //履约金额
            actualPerformanceAmount: params.actualPerformanceAmount, //实际履约金额
            performanceDateType: params.performanceDateType, //履约日期类型
        })
        formModalRef.value.validateFields(['finContractStagingIdList'])
    }

    onMounted(async () => {
        clearSelections(['settlementMode'])
        querySelections(['settlementMode'])
    })

    const confrimLastRecord = () => {
        Modal.confirm({
            icon: () => {},
            title: '复用上次记账内容',
            content: '复用内容，可通过问号图标来查看，您确定复用上次出纳记账内容吗？',
            onOk: () => {
                handleBookkeepingInfo('', {
                    amountType: modalTitle.value.includes('收入') ? 0 : 1,
                    subjectCode: activeMenuItem.value?.subjectCode,
                })
            },
        })
    }
    return {
        formData,
        formModel,
        showModal,
        modalTitle,
        confirmAdd,
        confirmClose,
        handleShowModal,
        editForm,
        loading,
        voucherUploadModalOpen,
        clickVoucher,
        currentRecord,
        submit,
        selectContract,
        currentAmountType,
        currentAddType,
        hasFinVoucherId,
        hasVoucher,
        confrimLastRecord,
    }
}

```

案例2：

```vue
<template>
	<FormModal
            ref="formModal"
            v-model:formData="Modal_formData"
            v-model:show="Modal_showModal"
            :formModel="Modal_formModel"
            :modal-title="Modal_isAddAsset ? '辅助核算类型新增' : '辅助核算类型编辑'"
            layout="vertical"
            modal-width="50vw"
            verticalGap="0"
            confirm-text="确定"
            labelAlign="right"
            :labelCol="{ style: { minWidth: '110px' } }"
            :cancel-text="'取消'"
            :okBtnLoading="Modal_loading"
            closable
            @confirm="Modal_confirmAdd"
        />
</template>


<script setup>
  import useSubsidiaryAccounting from './composables/useSubsidiaryAccounting'
  
  const {
    syncOpen,
    syncOk,
    selectedItems,
    activeMenuItem,
    searchColumn,
    searchFormData,
    rowSelection,
    modalformData: Modal_formData,
    modalshowModal: Modal_showModal,
    modalformModel: Modal_formModel,
    modalisAddAsset: Modal_isAddAsset,
    modalloading: Modal_loading,
    modalconfirmAdd: Modal_confirmAdd,
    tablelist: Table_list,
    tablecolumns: Table_columns,
    tableloading: Table_loading,
    tablepagination: Table_pagination,
    onAdd,
    onAllDelete,
    applicationLedger,
    getlist,
    refreshlist,
    clickMenu,
    configurationDataItems,
    onDelete,
    onEdit,
} = useSubsidiaryAccounting()
</script>
```

```js
// useSubsidiaryAccounting.js
import { createVNode, ref } from 'vue'
import { ExclamationCircleOutlined } from '@ant-design/icons-vue'
import { apis } from '@api'
import { useRouter } from 'vue-router'
import { message, Modal } from 'x-widgets'
import { getNodeByKeyDeep } from 'x-utils'
import { currentAccount } from '@/shared'

export default function useSubsidiaryAccounting({ accountTreeRef }) {
    const router = useRouter()
    const syncOpen = ref(false)
    const activeMenuItem = ref({})
    const baseAccountBookId = ref('')
    const searchColumn = ref([
        {
            el: 'input',
            label: '关键字',
            name: 'keyword',
            selfProps: {
                labelAlign: 'right',
                labelCol: { style: { minWidth: '0' } },
            },
            props: {
                placeholder: '请输入辅助核算类型关键字',
                allowClear: true,
                style: {
                    width: '250px',
                },
            },
        },
    ])
    const tableloading = ref(false)
    const searchFormData = reactive({
        keyword: '',
    })
    const tablelist = ref([])
    const tablecolumns = ref([
        {
            title: '序号',
            dataIndex: 'sort',
            key: 'sort',
            width: 150,
            align: 'center',
        },
        {
            title: '辅助核算类型',
            dataIndex: 'assistCalculateName',
            width: 150,
            align: 'center',
        },
        {
            title: '类型编号',
            dataIndex: 'calculateCode',
            width: 120,
            align: 'center',
        },
        {
            title: '操作',
            fixed: 'right',
            key: 'operation',
            width: 200,
            align: 'center',
        },
    ])
    const selectedRowKeys = ref([])
    const selectedItems = ref([])
    const rowSelection = {
        fixed: true,
        selectedRowKeys: selectedRowKeys,
        onChange: (_selectedRowKeys, _selectedItems) => {
            selectedRowKeys.value = _selectedRowKeys
            selectedItems.value = _selectedItems
        },
    }
    const tablepagination = reactive({
        current: 1,
        pageSize: 50,
        total: 0,
        showQuickJumper: true,
        showSizeChanger: true,
        showTotal: total => {
            return `共${total}条`
        },
        onChange: (_current, _size) => {
            tablepagination.current = _current
            tablepagination.pageSize = _size
            getAssistCalculateList()
        },
    })

    const syncClose = () => {
        syncOpen.value = false
        selectedItems.value = []
        selectedRowKeys.value = []
    }
    const syncOk = idList => {
        let basAssistCalculateId = selectedItems.value.map(item => item.baseAssistCalculateId)
        apis.accountManage
            .assistCalculateSynchronizeData({
                accountBookId: idList,
                basAssistCalculateId,
            })
            .then(res => {
                if (res.code == 200) {
                    message.success('数据应用成功！')
                    syncClose()
                }
            })
    }

    const getlist = () => {
        getAssistCalculateList()
    }

    const refreshlist = () => {
        getAssistCalculateList()
    }

    const menusList = computed(() => accountTreeRef.value?.menusList || [])
    const clickMenu = val => {
        activeMenuItem.value = getNodeByKeyDeep(val, menusList.value, {
            fieldKey: 'sysOrganizationId',
            children: 'children',
        })
        if (activeMenuItem.value && activeMenuItem.value.baseAccountBookId) {
            baseAccountBookId.value = activeMenuItem.value?.baseAccountBookId
            getAssistCalculateList()
        } else {
            tablelist.value = []
        }
    }
    const onAdd = () => {
        resetData()
        modalshowModal.value = true
        modalisAddAsset.value = true
    }

    const onDelete = val => {
        Modal.confirm({
            title: '删除',
            content: `您确定删除${val.assistCalculateName}吗？`,
            icon: createVNode(ExclamationCircleOutlined),
            okText: '删除',
            cancelText: '取消',
            onOk() {
                deleteAssistCalculateById([
                    {
                        baseAssistCalculateId: val.baseAssistCalculateId,
                        operateDeleted: val.operateDeleted,
                    },
                ])
            },
        })
    }

    const onAllDelete = () => {
        if (selectedRowKeys.value.length > 0) {
            Modal.confirm({
                title: '删除',
                content: `您确定删除所选择的辅助核算类型吗？`,
                icon: createVNode(ExclamationCircleOutlined),
                okText: '删除',
                cancelText: '取消',
                onOk() {
                    deleteAssistCalculateById(
                        selectedItems.value.map(item => {
                            return {
                                baseAssistCalculateId: item.baseAssistCalculateId,
                                operateDeleted: item.operateDeleted,
                            }
                        }),
                    )
                },
            })
        } else {
            message.warning('请选择需要操作的列表数据')
        }
    }

   
    const modalshowModal = ref(false)
    // 表单字段
    const validateAccountName = async (rule, value) => {
        if (!value) {
            return Promise.reject('请输入')
        } else if (/\s/g.test(value)) {
            return Promise.reject('数据类型名称不能有空格符，请删除空格')
        } else {
            return Promise.resolve()
        }
    }
    const column = {
        column: [
            {
                el: 'input',
                label: '辅助核算类型',
                name: 'assistCalculateName',
                rules: [{ required: true, validator: validateAccountName }],
                selfProps: {
                    extra: '不能超过20个字，不能有空格,不能重复',
                },
                props: {
                    maxlength: 20,
                    allowClear: true,
                },
            },
        ],
    }
    const modalformModel = ref([column])
    const modalisAddAsset = ref()
    const modalloading = ref(false)

   

    return {
        modalformData,
        modalshowModal,
        modalformModel,
        modalisAddAsset,
        modalloading,
        syncOpen,
        selectedItems,
        activeMenuItem,
        searchColumn,
        searchFormData,

    }
}

```

