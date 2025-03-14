# Table 表格

展示行列数据。

## 何时使用 [#](https://www.antdv.com/components/table-cn#何时使用)

- 当有大量结构化的数据需要展现时；
- 当需要对数据进行排序、搜索、分页、自定义操作等复杂行为时。

## 如何使用 [#](https://www.antdv.com/components/table-cn#如何使用)

指定表格的数据源 `dataSource` 为一个数组。

```vue
<template>
  <x-table :dataSource="dataSource" :columns="columns" />
</template>
<script>
  export default {
    setup() {
      return {
        dataSource: [
          {
            key: '1',
            name: '胡彦斌',
            age: 32,
            address: '西湖区湖底公园1号',
          },
          {
            key: '2',
            name: '胡彦祖',
            age: 42,
            address: '西湖区湖底公园1号',
          },
        ],

        columns: [
          {
            title: '姓名',
            dataIndex: 'name',
            key: 'name',
          },
          {
            title: '年龄',
            dataIndex: 'age',
            key: 'age',
          },
          {
            title: '住址',
            dataIndex: 'address',
            key: 'address',
          },
        ],
      };
    },
  };
</script>
```

## Table API [#](https://www.antdv.com/components/table-cn#table)

| 参数                     | 说明                                                         | 类型                                                         | 默认值                                                       | 版本  |
| :----------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :---- |
| bodyCell                 | 个性化单元格                                                 | v-slot:bodyCell="{text, record, index, column}"              | -                                                            | 3.0   |
| bordered                 | 是否展示外边框和列边框                                       | boolean                                                      | false                                                        |       |
| childrenColumnName       | 指定树形结构的列名                                           | string                                                       | `children`                                                   |       |
| columns                  | 表格列的配置描述，具体项见[下表](https://www.antdv.com/components/table-cn#column) | array                                                        | -                                                            |       |
| components               | 覆盖默认的 table 元素                                        | object                                                       | -                                                            |       |
| customFilterDropdown     | 自定义筛选菜单，需要配合 `column.customFilterDropdown` 使用  | v-slot:customFilterDropdown="[FilterDropdownProps](https://www.antdv.com/components/table-cn#filterdropdownprops)" | -                                                            | 3.0   |
| customFilterIcon         | 自定义筛选图标                                               | v-slot:customFilterIcon="{filtered, column}"                 | -                                                            | 3.0   |
| customHeaderRow          | 设置头部行属性                                               | Function(columns, index)                                     | -                                                            |       |
| customRow                | 设置行属性                                                   | Function(record, index)                                      | -                                                            |       |
| dataSource               | 数据数组                                                     | object[]                                                     |                                                              |       |
| defaultExpandAllRows     | 初始时，是否展开所有行                                       | boolean                                                      | false                                                        |       |
| defaultExpandedRowKeys   | 默认展开的行                                                 | string[]                                                     | -                                                            |       |
| emptyText                | 自定义空数据时的显示内容                                     | v-slot:emptyText                                             | -                                                            | 3.0   |
| expandedRowKeys(v-model) | 展开的行，控制属性                                           | string[]                                                     | -                                                            |       |
| expandedRowRender        | 额外的展开行                                                 | Function(record, index, indent, expanded):VNode \| v-slot:expandedRowRender="{record, index, indent, expanded}" | -                                                            |       |
| expandFixed              | 控制展开图标是否固定，可选 true `left` `right`               | boolean \| string                                            | false                                                        | 3.0   |
| expandColumnTitle        | 自定义展开列表头                                             | v-slot                                                       | -                                                            | 4.0.0 |
| expandIcon               | 自定义展开图标                                               | Function(props):VNode \| v-slot:expandIcon="props"           | -                                                            |       |
| expandRowByClick         | 通过点击行来展开子行                                         | boolean                                                      | `false`                                                      |       |
| footer                   | 表格尾部                                                     | Function(currentPageData)\|v-slot:footer="currentPageData"   |                                                              |       |
| getPopupContainer        | 设置表格内各类浮层的渲染节点，如筛选菜单                     | (triggerNode) => HTMLElement                                 | `() => TableHtmlElement`                                     | 1.5.0 |
| headerCell               | 个性化头部单元格                                             | v-slot:headerCell="{title, column}"                          | -                                                            | 3.0   |
| indentSize               | 展示树形数据时，每层缩进的宽度，以 px 为单位                 | number                                                       | 15                                                           |       |
| loading                  | 页面是否加载中                                               | boolean\|[object](https://www.antdv.com/components/spin-cn)  | false                                                        |       |
| locale                   | 默认文案设置，目前包括排序、过滤、空数据文案                 | object                                                       | filterConfirm: `确定` filterReset: `重置` emptyText: `暂无数据` |       |
| pagination               | 分页器，参考[配置项](https://www.antdv.com/components/table-cn#pagination)或 [pagination](https://www.antdv.com/components/pagination-cn/)文档，设为 false 时不展示和进行分页 | object \| `false`                                            |                                                              |       |
| rowClassName             | 表格行的类名                                                 | Function(record, index):string                               | -                                                            |       |
| rowExpandable            | 设置是否允许行展开                                           | (record) => boolean                                          | -                                                            | 3.0   |
| rowKey                   | 表格行 key 的取值，可以是字符串或一个函数                    | string\|Function(record):string                              | 'key'                                                        |       |
| rowSelection             | 列表项是否可选择，[配置项](https://www.antdv.com/components/table-cn#rowselection) | object                                                       | null                                                         |       |
| scroll                   | 表格是否可滚动，也可以指定滚动区域的宽、高，[配置项](https://www.antdv.com/components/table-cn#scroll) | object                                                       | -                                                            |       |
| showExpandColumn         | 设置是否展示行展开列                                         | boolean                                                      | true                                                         | 3.0   |
| showHeader               | 是否显示表头                                                 | boolean                                                      | true                                                         |       |
| showSorterTooltip        | 表头是否显示下一次排序的 tooltip 提示。当参数类型为对象时，将被设置为 Tooltip 的属性 | boolean \| [Tooltip props](https://www.antdv.com/components/tooltip/) | true                                                         | 3.0   |
| size                     | 表格大小                                                     | `large` | `middle` | `small`                                 | `large`                                                      |       |
| sortDirections           | 支持的排序方式，取值为 `ascend` `descend`                    | Array                                                        | [`ascend`, `descend`]                                        |       |
| sticky                   | 设置粘性头部和滚动条                                         | boolean \| `{offsetHeader?: number, offsetScroll?: number, getContainer?: () => HTMLElement}` | -                                                            | 3.0   |
| summary                  | 总结栏                                                       | v-slot:summary                                               | -                                                            | 3.0   |
| tableLayout              | 表格元素的 [table-layout](https://developer.mozilla.org/zh-CN/docs/Web/CSS/table-layout) 属性，设为 `fixed` 表示内容不会影响列的布局 | - \| 'auto' \| 'fixed'                                       | 无固定表头/列或使用了 `column.ellipsis` 时，默认值为 `fixed` | 1.5.0 |
| title                    | 表格标题                                                     | Function(currentPageData)\|v-slot:title="currentPageData"    |                                                              |       |
| transformCellText        | 数据渲染前可以再次改变，一般用于空数据的默认配置，可以通过 [ConfigProvider](https://www.antdv.com/components/config-provider-cn/) 全局统一配置 | Function({ text, column, record, index }) => any，此处的 text 是经过其它定义单元格 api 处理后的数据，有可能是 VNode \| string \| number 类型 | -                                                            | 1.5.4 |

## 事件 [#](https://www.antdv.com/components/table-cn#事件)

| 事件名称           | 说明                       | 回调参数                                                     |
| :----------------- | :------------------------- | :----------------------------------------------------------- |
| change             | 分页、排序、筛选变化时触发 | Function(pagination, filters, sorter, { action, currentDataSource }) |
| expand             | 点击展开图标时触发         | Function(expanded, record)                                   |
| expandedRowsChange | 展开的行变化时触发         | Function(expandedRows)                                       |
| resizeColumn       | 拖动列时触发               | Function(width, column)                                      |

## Column [#](https://www.antdv.com/components/table-cn#column)

列描述数据对象，是 columns 中的一项，Column 使用相同的 API。

| 参数                              | 说明                                                         | 类型                                                         | 默认值                  | 版本                       |
| :-------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :---------------------- | :------------------------- |
| align                             | 设置列的对齐方式                                             | `left` | `right` | `center`                                  | `left`                  |                            |
| colSpan                           | 表头列合并,设置为 0 时，不渲染                               | number                                                       |                         |                            |
| customCell                        | 设置单元格属性                                               | Function(record, rowIndex, column)                           | -                       | column add from 3.0        |
| customFilterDropdown              | 启用 v-slot:customFilterDropdown，优先级低于 filterDropdown  | boolean                                                      | false                   | 3.0                        |
| customHeaderCell                  | 设置头部单元格属性                                           | Function(column)                                             | -                       |                            |
| customRender                      | 生成复杂数据的渲染函数，参数分别为当前行的值，当前行数据，行索引 | Function({text, record, index, column}) {}                   | -                       |                            |
| dataIndex                         | 列数据在数据项中对应的路径，支持通过数组查询嵌套路径         | string \| string[]                                           | -                       |                            |
| defaultFilteredValue              | 默认筛选值                                                   | string[]                                                     | -                       | 1.5.0                      |
| filterResetToDefaultFilteredValue | 点击重置按钮的时候，是否恢复默认筛选值                       | boolean                                                      | false                   | 3.3.0                      |
| defaultSortOrder                  | 默认排序顺序                                                 | `ascend` | `descend`                                         | -                       |                            |
| ellipsis                          | 超过宽度将自动省略，暂不支持和排序筛选一起使用。 设置为 `true` 或 `{ showTitle?: boolean }` 时，表格布局将变成 `tableLayout="fixed"`。 | boolean \| { showTitle?: boolean }                           | false                   | 3.0                        |
| filterDropdown                    | 可以自定义筛选菜单，此函数只负责渲染图层，需要自行编写各种交互 | VNode \| (props: FilterDropdownProps) => VNode               | -                       |                            |
| filterDropdownOpen                | 用于控制自定义筛选菜单是否可见                               | boolean                                                      | -                       |                            |
| filtered                          | 标识数据是否经过过滤，筛选图标会高亮                         | boolean                                                      | false                   |                            |
| filteredValue                     | 筛选的受控属性，外界可用此控制列的筛选状态，值为已筛选的 value 数组 | string[]                                                     | -                       |                            |
| filterIcon                        | 自定义 filter 图标。                                         | VNode \| ({filtered: boolean, column: Column}) => vNode      | false                   |                            |
| filterMode                        | 指定筛选菜单的用户界面                                       | 'menu' \| 'tree'                                             | 'menu'                  | 3.0                        |
| filterMultiple                    | 是否多选                                                     | boolean                                                      | true                    |                            |
| filters                           | 表头的筛选菜单项                                             | object[]                                                     | -                       |                            |
| filterSearch                      | 筛选菜单项是否可搜索                                         | boolean \| function(input, filter):boolean                   | false                   | boolean:3.0 function:3.3.0 |
| fixed                             | 列是否固定，可选 `true`(等效于 left) `'left'` `'right'`      | boolean\|string                                              | false                   |                            |
| key                               | Vue 需要的 key，如果已经设置了唯一的 `dataIndex`，可以忽略这个属性 | string                                                       | -                       |                            |
| maxWidth                          | 拖动列最大宽度，会受到表格自动调整分配宽度影响               | number                                                       | -                       | 3.0                        |
| minWidth                          | 拖动列最小宽度，会受到表格自动调整分配宽度影响               | number                                                       | 50                      | 3.0                        |
| resizable                         | 是否可拖动调整宽度，此时 width 必须是 number 类型            | boolean                                                      | -                       | 3.0                        |
| responsive                        | 响应式 breakpoint 配置列表。未设置则始终可见。               | [Breakpoint](https://www.antdv.com/components/table-cn#breakpoint)[] | -                       | 3.0                        |
| rowScope                          | 设置列范围                                                   | `row` | `rowgroup`                                           | -                       | 4.0                        |
| showSorterTooltip                 | 表头显示下一次排序的 tooltip 提示, 覆盖 table 中 `showSorterTooltip` | boolean \| [Tooltip props](https://www.antdv.com/components/tooltip/#api) | true                    |                            |
| sortDirections                    | 支持的排序方式，取值为 `'ascend'` `'descend'`                | Array                                                        | `['ascend', 'descend']` | 1.5.0                      |
| sorter                            | 排序函数，本地排序使用一个函数(参考 [Array.sort](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) 的 compareFunction)，需要服务端排序可设为 true | Function\|boolean                                            | -                       |                            |
| sortOrder                         | 排序的受控属性，外界可用此控制列的排序，可设置为 `'ascend'` `'descend'` `null` | string                                                       | -                       |                            |
| title                             | 列头显示文字                                                 | string                                                       | -                       |                            |
| width                             | 列宽度                                                       | string\|number                                               | -                       |                            |
| onFilter                          | 本地模式下，确定筛选的运行函数, 使用 template 或 jsx 时作为`filter`事件使用 | Function                                                     | -                       |                            |
| onFilterDropdownOpenChange        | 自定义筛选菜单可见变化时调用，使用 template 或 jsx 时作为`filterDropdownOpenChange`事件使用 | function(open) {}                                            | -                       | 4.0                        |

## 实际案例

```vue
<template>
    <Table
        :data-source="tableData"
        :columns="tableColumns"
        size="small"
        striped
        :scroll="{ x: 300 }"
        :loading="loading"
    >
        <template #bodyCell="{ record, column, index }">
            <template v-if="column.dataIndex == 'operation'">
                <Button type="link" @click="chooseAction(record.finBudgetManageId)">选择</Button>
            </template>
        </template>
    </Table>
</template>
<script setup>
import { Table, Button } from 'x-widgets'
import useTable from './composable/useTable'

const { tableColumns, tableData, chooseAction, loading } = useTable()
</script>
<style lang="less" scoped></style>

```

```js
// useTable.js
import { apis } from '@/api'
import { onActivated, ref } from 'vue'
import { useRouter } from 'vue-router'
import { useApi } from 'x-utils'

export default function useTable() {
    const tableData = ref([])

    const tableColumns = [
        {
            title: '序号',
            dataIndex: 'sort',
            width: 80,
            customRender: ({ text, index }) => {
                return index + 1
            },
        },
        {
            title: '预算名称',
            dataIndex: 'budgetName',
        },
        {
            title: '预算年份',
            dataIndex: 'budgetYear',
        },
        {
            title: '审批状态',
            dataIndex: 'approvalStatusText',
        },
        {
            title: '编制人',
            dataIndex: 'compiler',
        },
        {
            title: '编制时间',
            dataIndex: 'preparedDate',
        },
        {
            title: '操作',
            width: 100,
            dataIndex: 'operation',
        },
    ]

    tableColumns.forEach(item => {
        item.align = 'center'
    })

    const pagination = reactive({
        current: 1,
        pageSize: 10,
        total: 0,
        showSizeChanger: true,
        showQuickJumper: true,
        showTotal: total => `共 ${total} 条数据`,
        onChange: (page, size) => {
            pagination.current = page
            pagination.pageSize = size
            getTableData()
        },
    })

    const loading = ref(false)

    const getTableData = async () => {
        const api = useApi(apis.budgetManage.getAdjustableFinBudgetPage)
        loading.value = true
        await api.fetch({
            ...pagination,
            size: pagination.pageSize,
        })
        loading.value = api.loading.value

        if (api.error.value) return
        tableData.value = api.data.value.records
        pagination.total = parseInt(api.data.value.total)
    }

    onActivated(() => {
        getTableData()
    })

    const router = useRouter()
    const chooseAction = key => {
        router.push(`/budgetManage/financialPreparation?id=${key}&type=adjust`)
    }

    return {
        tableColumns,
        tableData,
        chooseAction,
        loading,
    }
}

```

