# FunctionButton 组件使用文档

组件概述：FunctionButton组件提供导出、下载、上传、预览等功能，通过传入配置项实现不同的功能。

## 组件源码

```tsx
/**
 * @description: 功能按钮组件，提供导出、下载、上传、预览等功能
 * @name FunctionButton
 * @author WuShiLi
 */

import { defineComponent, computed, ref, Component, unref } from 'vue'
import { CloseOutlined } from '@ant-design/icons-vue'
import { useHandleFile, getUID } from 'x-utils'
import { message as AMessage, Button, Upload } from '../../index'
import { UploadDragger } from 'ant-design-vue'
import { FunctionButtonOptions } from './types/props'
import type { FunctionButtonProps, FunctionButtonOptionsType } from './types/props'
import style from './style/index.module.less'

export default defineComponent<FunctionButtonProps>({
    name: 'FunctionButton',
    emits: ['update:fileList', 'uploadFile', 'success', 'error', 'progress', 'uploading'],
    props: {
        ...Button.props,
        ...FunctionButtonOptions(),
    },
    setup(props, { attrs, slots, emit, expose }) {
        const defaultOption: FunctionButtonOptionsType = {
            uploadComponent: 'Upload',
            btnText: '',
            exportAPI: null, // 导出API函数
            exportParams: {},
            uploadURL: '', //导入/上传api地址
            uploadVideoURL: '', //导入/上传视频api地址
            uploadData: {}, //上传的data参数
            uploadHeaders: {}, //上传的headers参数
            uploadAcceptTypes: '.xls, .xlsx', //上传的格式
            downloadURL: '', //下载地址
            previewURL: '', //预览地址
            fileName: '', //下载文件名称
            fileType: '', //下载文件格式/类型
            imgMaxSize: 100, //图片最大大小
            fileMaxSize: 25, //文件最大大小
            videoMaxSize: 50, //视频最大大小
            showProgress: false, //是否显示上传进度
            printAPI: '', // 打印字符串API（打印和导出功能exportParams可复用）
            imgMinSize: 0, //图片最小大小 以k为单位
            videoMinSize: 0, //视频最小大小
            pdfMinSize: 0, //pdf最小大小
            wordMinSize: 0, //word最小大小
            excelMinSize: 0, //excel最小大小
            fileMinSize: 0, //文件最小大小
            validateFilename: false, // 是否需要校验文件名
            onExportSuccess: () => {}, // 导出成功回调
            onExportError: () => {}, // 导出失败回调
        }

        const options = Object.assign(defaultOption, props.options)

        const { maxCount, showProgress, progressText } = options || {}

        const {
            isExporting,
            exportFile,
            uploadFile,
            downloadFile,
            previewFile,
            imageType,
            videoType,
            pdfType,
            wordType,
            excelType,
            otherType,
        } = useHandleFile({
            OBS_FILE_DOMAIN: options.OBS_FILE_DOMAIN,
            emits: emit,
        })

        const innerFileList = computed<Array<any>>({
            get() {
                return props.fileList || []
            },
            set(val: Array<any>) {
                if (val.length > maxCount) {
                    AMessage.error(`最多上传${maxCount}个文件`)
                    return
                }
                emit('update:fileList', val)
            },
        })
        // 导出
        function handleExport(params: any) {
            if (!options.exportAPI) return AMessage.error('导出API未配置！')
            let reqParams = params
            if (typeof params == 'function') {
                reqParams = params()
            }
            exportFile(options.exportAPI, {
                downloadName: options.fileName,
                fileType: options.fileType,
                params: reqParams,
                onExportSuccess: options.onExportSuccess,
                onExportError: options.onExportError,
            })
        }

        // 下载
        function handleDownload() {
            downloadFile(options.downloadURL)
        }

        // 预览
        function handlePreview() {
            previewFile(options.previewURL, { fileType: options.fileType })
        }

        const finalUploadURL = ref(options.uploadURL || options.uploadVideoURL)
        /** 上传前校验 */
        const beforeUpload = (file, files) => {
            const { name } = file
            if (options.validateFilename) {
                const regex = /[\/:*?"<>|]/g
                if (regex.test(name)) {
                    AMessage.error('文件名不能包含特殊字符！')
                    return Promise.reject()
                }
            }
            const fileType = name.split('.').pop().toLowerCase()
            if (videoType.includes(fileType)) {
                finalUploadURL.value = options.uploadVideoURL
            } else {
                finalUploadURL.value = options.uploadURL
            }
            //检测文件类型
            if (options.uploadAcceptTypes) {
                const acceptTypes = options.uploadAcceptTypes
                    .split(',')
                    .map(item => item.trim().replace('.', ''))

                if (!acceptTypes.includes(fileType)) {
                    AMessage.error('文件类型不支持！')
                    return Promise.reject()
                }
            }

            // 校验文件数是否超过限制
            if (maxCount) {
                const leftCount = computed(() => maxCount - innerFileList.value?.length)
                const index = files.findIndex(elem => elem === file)

                if (index >= leftCount.value) {
                    AMessage.error('超过文件数量限制！')
                    // innerFileList.value = files.slice(0, maxCount)
                    return Promise.reject()
                }
            }
            // 校验文件大小
            const fileNameArr = file.name.split('.') // 获取文件后缀
            const suffix = fileNameArr[fileNameArr.length - 1]
            let isLessMax = file.size / 1024 / 1024 <= options.fileMaxSize,
                maxSize = options.fileMaxSize
            let isLessMinSize = file.size / 1024 < options.fileMinSize,
                minSize = options.fileMinSize
            let typeArr = [
                {
                    type: imageType,
                    minSize: options.imgMinSize,
                    maxSize: options.imgMaxSize,
                },
                {
                    type: videoType,
                    minSize: options.videoMinSize,
                    maxSize: options.videoMaxSize,
                },
                {
                    type: pdfType,
                    minSize: options.pdfMinSize,
                    maxSize: options.pdfMaxSize,
                },
                {
                    type: wordType,
                    minSize: options.wordMinSize,
                    maxSize: options.wordMaxSize,
                },
                {
                    type: excelType,
                    minSize: options.excelMinSize,
                    maxSize: options.excelMaxSize,
                },
                {
                    type: otherType,
                    minSize: options.excelMinSize,
                    maxSize: options.excelMaxSize,
                },
            ]
            for (let i = 0; i < typeArr.length; i++) {
                const el = typeArr[i]
                // 最大 以MB为单位
                if (el.type.includes(suffix) && el.maxSize) {
                    isLessMax = file.size / 1024 / 1024 <= el.maxSize
                    maxSize = el.maxSize
                }
                // 最小 以k为单位
                if (el.type.includes(suffix) && el.minSize) {
                    isLessMinSize = file.size / 1024 < el.minSize
                    minSize = el.minSize
                }
            }
            if (!isLessMax) {
                if (imageType.includes(suffix)) {
                    AMessage.error(`图片大小不能超过 ${maxSize}MB!`)
                } else {
                    AMessage.error(`文件大小不能超过 ${maxSize}MB!`)
                }
                return Promise.reject()
            }

            if (isLessMinSize) {
                if (imageType.includes(suffix)) {
                    AMessage.error(`图片大小不能小于 ${minSize}k!`)
                } else if (videoType.includes(suffix)) {
                    AMessage.error(`视频大小不能小于 ${minSize}k!`)
                } else {
                    AMessage.error(`文件大小不能小于 ${minSize}k!`)
                }
                return Promise.reject()
            }
            return true
        }
        const uploadPercent = ref(0)
        const _progressShow = ref(false)
        function handleProgress(percent, _file) {
            uploadPercent.value = percent
        }

        const UID = getUID()

        function clickUpload(opts: any = {}) {
            if (opts && typeof opts.validateFilename === 'boolean') {
                options.validateFilename = opts.validateFilename
            }
            const node: HTMLElement = document.querySelector(`#${UID}`)
            node.click()
        }

        const fileRequestMap = new Map()

        // 拖拽上传-全部取消上传
        function abortUploadAll(e) {
            e.stopPropagation()
            fileRequestMap.forEach(xhr => {
                xhr.abort()
            })
            _progressShow.value = false
            uploadPercent.value = 0
        }

        // 取消上传
        function abortUpload(uid) {
            const xhr = fileRequestMap.get(uid)
            xhr?.abort()
        }

        // 自定义上传
        function _customRequest(options) {
            const { file, data, action, headers, onSuccess, onError, onProgress } = options
            const xhr = new XMLHttpRequest()
            //监听上传进度条
            xhr.upload.onprogress = function (e) {
                onProgress &&
                    onProgress({
                        percent: (e.loaded / e.total) * 100,
                        loaded: e.loaded,
                        total: e.total,
                        file,
                    })
            }
            const formData = new FormData()
            formData.append('file', file)
            // 额外的上传数据
            if (data) {
                Object.keys(data).forEach(key => {
                    formData.append(key, data[key])
                })
            }
            xhr.open('POST', action, true)
            // 设置请求头
            if (headers) {
                Object.keys(headers).forEach(key => {
                    xhr.setRequestHeader(key, headers[key])
                })
            }
            // 监听上传成功或失败的处理
            xhr.onload = () => {
                if (xhr.status >= 200 && xhr.status < 300) {
                    const response = JSON.parse(xhr.responseText)
                    onSuccess(response, xhr)
                } else {
                    onError && onError(new Error('Upload failed'), xhr)
                }
            }
            xhr.send(formData)
            if (!fileRequestMap.has(file.uid)) {
                fileRequestMap.set(file.uid, xhr)
            }
        }
        function gotoPrint() {
            let exportParams = {}
            if (typeof options.exportParams === 'function') {
                exportParams = options.exportParams()
            } else {
                exportParams = options.exportParams
            }
            const params = {
                exportAPI: options.printAPI,
                ...exportParams,
            }
            // 把params转成url参数
            let str = ''
            for (const key in params) {
                if (params[key]) {
                    str += `${key}=${params[key]}&`
                }
            }
            if (options.printAPI) {
                const uid = getUID()
                const _printExportParams = JSON.parse(
                    localStorage.getItem('_printExportParams') || '{}',
                )
                localStorage.setItem(
                    '_printExportParams',
                    JSON.stringify({
                        [uid]: JSON.stringify(exportParams),
                        ..._printExportParams,
                    }),
                )
                // 跳转到打印页面 /financeBookPrint 拼接params
                // window.open(`/financeBookPrint?uid=${uid}&${str}`, '_blank')
                const ID = 'print-iframe'
                const $iframeEl = document.querySelector('#' + ID)
                if ($iframeEl) {
                    document.body.removeChild($iframeEl)
                }
                const $iframe = document.createElement('iframe')
                $iframe.id = ID
                $iframe.src = `/financeBookPrint?uid=${uid}&${str}`
                $iframe.style.display = 'none'
                document.body.insertAdjacentElement('beforeend', $iframe)
            }
        }

        expose({
            innerFileList,
            emit,
            clickUpload,
            abortUpload,
            loading: isExporting,
        })
        return () => {
            const uploadComponnets = {
                Upload,
                UploadDragger,
            }
            const type = options.uploadComponent || 'Upload'
            const Component = uploadComponnets[type]
            const renderComponents: Component[] | any = []

            const ExportButton = () =>
                options.exportAPI ? (
                    <Button
                        {...attrs}
                        {...props}
                        loading={isExporting.value}
                        onClick={() => handleExport((props.options as any).exportParams)}
                    >
                        {slots.default ? slots.default() : unref(options.btnText) || '导出'}
                    </Button>
                ) : null

            const DownloadButton = () =>
                options.downloadURL ? (
                    <Button {...attrs} {...props} onClick={handleDownload}>
                        {slots.default ? slots.default() : unref(options.btnText) || '下载'}
                    </Button>
                ) : null

            const PreviewButton = () =>
                options.previewURL ? (
                    <Button {...attrs} {...props} onClick={handlePreview}>
                        {slots.default ? slots.default() : unref(options.btnText) || '预览'}
                    </Button>
                ) : null

            const UploadButton = () =>
                options.uploadURL || options.uploadVideoURL ? (
                    <Component
                        id={UID}
                        {...attrs}
                        name="file"
                        accept={options.uploadAcceptTypes}
                        action={finalUploadURL.value}
                        data={options.uploadData}
                        headers={options.uploadHeaders}
                        beforeUpload={beforeUpload}
                        showUploadList={false}
                        multiple={options.maxCount > 1}
                        withCredentials={false}
                        onChange={({ file, fileList }) => {
                            emit('uploadFile', { file, fileList, btnkey: props.btnkey })
                            const _file = uploadFile({ file, fileList }, props.btnkey)
                            if (_file.status === 'done') {
                                uploadPercent.value = 0
                                _progressShow.value = false
                            } else if (_file.status === 'uploading') {
                                _progressShow.value = true
                            } else {
                                _progressShow.value = false
                            }
                        }}
                        onProgress={({ percent, file }) => {
                            emit('progress', percent, file)
                            handleProgress(percent, file)
                        }}
                        customRequest={_customRequest}
                    >
                        {slots.default ? (
                            <div class={style.progressWrap}>
                                {uploadPercent.value > 0 &&
                                    showProgress &&
                                    _progressShow.value &&
                                    slots.progress && (
                                        <div class={style.progressBarWrap}>
                                            <div
                                                class={style.progressBar}
                                                style={{ width: `${uploadPercent.value}%` }}
                                            ></div>
                                            <div class={style.progressText}>
                                                {slots.progress ? (
                                                    slots.progress({
                                                        percent: uploadPercent.value,
                                                    })
                                                ) : (
                                                    <span>{progressText}</span>
                                                )}
                                            </div>
                                            <CloseOutlined
                                                class={style.progressDeleteIcon}
                                                onClick={e => abortUploadAll(e)}
                                                title={'取消上传'}
                                            />
                                        </div>
                                    )}

                                {slots.default()}
                            </div>
                        ) : (
                            <Button
                                loading={props.loading}
                                {...attrs}
                                {...props}
                                disabled={props.loading || props.disabled}
                            >
                                {attrs.btnText || options.btnText || '上传'}
                            </Button>
                        )}
                    </Component>
                ) : null

            const PrintButton = () =>
                options.printAPI ? (
                    <Button
                        {...attrs}
                        {...props}
                        loading={isExporting.value}
                        onClick={() => gotoPrint()}
                    >
                        {unref(options.btnText) || '打印'}
                    </Button>
                ) : null

            renderComponents.push(
                ExportButton(),
                DownloadButton(),
                PreviewButton(),
                UploadButton(),
                PrintButton(),
            )
            const components = renderComponents.filter(v => v)

            return components ? components[0] : <div></div>
        }
    },
})

```

组件的props:

```ts
import type { ButtonProps } from 'ant-design-vue'
import { PropType, ExtractPropTypes } from 'vue'

export interface FunctionButtonOptionsType {
    uploadComponent: 'Upload' | 'UploadDragger'
    btnText?: string
    exportAPI?: () => Promise<any> | null // 导出API函数
    exportParams?: any // 导出参数
    uploadURL?: string //导入/上传普通文件/图片api地址
    uploadVideoURL?: string //导入/上传视频api地址
    uploadData?: object //上传的data参数
    uploadHeaders?: object // 上传的headers参数
    uploadAcceptTypes?: string //上传的格式
    downloadURL?: string //下载地址
    previewURL?: string //预览地址
    fileName?: string //下载文件名称
    fileType?: string //下载文件格式/类型
    imgMaxSize?: number //图片最大大小
    videoMaxSize?: number //视频最大大小
    pdfMaxSize?: number //pdf最大大小
    wordMaxSize?: number //word最大大小
    excelMaxSize?: number //excel最大大小
    fileMaxSize?: number //文件最大大小

    imgMinSize?: number //图片最小大小 以k为单位
    videoMinSize?: number //视频最小大小
    pdfMinSize?: number //pdf最小大小
    wordMinSize?: number //word最小大小
    excelMinSize?: number //excel最小大小
    fileMinSize?: number //文件最小大小
    maxCount?: number //最大文件数
    OBS_FILE_DOMAIN?: string //OBS文件域名
    showProgress?: boolean //是否显示上传进度
    progressText?: string //上传进度文字
    printAPI?: string // 打印字符串API（打印和导出功能exportParams可复用）
    validateFilename?: boolean // 是否需要校验文件名
    onExportSuccess?: (res: any) => void // 导出成功回调
    onExportError?: (err: any) => void // 导出失败回调
}

export const FunctionButtonOptions = () => {
    return {
       // FunctionButton需要传入options配置
        options: {
            type: Object as () => PropType<FunctionButtonOptionsType>,
            default: () => ({
                btnText: '',
                exportAPI: '',
                exportParams: null,
                uploadURL: '',
                uploadData: {},
                uploadAcceptTypes: '.xls, .xlsx',
                downloadURL: '',
                previewURL: '',
                fileName: '',
                fileType: '',
                imgMaxSize: 10,
                fileMaxSize: 25,
                maxCount: null,
                OBS_FILE_DOMAIN: '',
            }),
        },
        // 文件上传模式时绑定的文件列表
        fileList: {
            type: Array,
            default: [],
        },
        // 按钮key
        btnkey: {
            type: [String, Number, Object],
            default: '',
        },
    }
}
export type FunctionButtonOptionsProps = Partial<
    ExtractPropTypes<ReturnType<typeof FunctionButtonOptions>>
>

export type FunctionButtonProps = ButtonProps & FunctionButtonOptionsProps

```

## 使用案例

### 案例一

```vue
<template>
  <FunctionButton
         type="primary"
         :options="exportButtonOptions"
         class="flex justify-end mb-2 btn"
  />
</template>

<script setup>
 import { apis } from '@/api'
 const currentLevel = ref(0)
const exportButtonOptions = {
    exportAPI: () => {
        let api = null
        if (currentLevel.value >= 4) {
            if (currentType.value === 0) {
                api = apis.accountStatistics.exportStreetAssetFromCore
            } else if (currentType.value === 1) {
                api = apis.accountStatistics.exportStreetAssetFromCity
            }
        } else {
            api = apis.accountStatistics.exportAssetCardEachMonth_province
        }
        if (api) {
            return api({ ...getParams(currentParams.value) })
        }
    },
    btnText: '导出',
}
</script>
```

### 案例二

```vue
<template>
  <FunctionButton type="primary" :options="exportOptions" />
</template>
<script setup>
  import { apis } from '@/api'
    const exportOptions = {
        exportAPI: apis.accountStatistics.exportRegionProfitDistributionForm,
        exportParams: () => getParams(),
        btnText: '导出',
    }
</script>

```

