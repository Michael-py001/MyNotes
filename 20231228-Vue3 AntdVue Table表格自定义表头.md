# 20231228-Vue3 AntdVue Table 表格自定义表头

> antd vue 版本： 2.2.8 

在 colums 中设置 slots.title，定义该列的自定义表头插槽名称

```js
 {
        // title: '闲置资产(宗)', 注意这里title要注释，不然插槽不生效
        dataIndex: 'freeAssetNumAndUnit',
        align: 'center',
        width: 110,
        slots: {
            title: 'freeAssetNumAndUnit_title',
        },
    },
```

> **注意：这里title要注释，不然插槽不生效**

Vue3 jsx 语法：插槽编写

```jsx
<Table
    dataSource={list.value}
    columns={columns.value}
    pagination={false}
    scroll={{ x: '800', y: tableHeight.value }}
    class="MapDetailPopup_table"
    bordered={false}
    customRow={(record, index) => {
        return {
            onClick: event => {
                clickDetail(record)
            },
        }
    }}
    >
    {{
        freeAssetNumAndUnit_title: ({
            record,
            text,
        }) => {
            return (
                <SorterHead columnKey="freeAssetNumAndUnit">
                    <span>闲置资产(宗)</span>
                </SorterHead>
            )
        },
    }}
</Table>
```

效果：

![image-20231229102420682](https://s2.loli.net/2023/12/29/p8nGkEBVu6eDN7w.png)
