# 20240521-antd-vue Table表格滚动到指定行

## 实现效果

![企业微信截图_17162731259390(1)](https://s2.loli.net/2024/05/21/KAlZDn7Thby5QwU.png)

## 实现思路

通过DOM操作，首先获取到Table元素，再通过改行数据的key值，从table中找到对应的tr。

得到该tr的滚动所在高度，也就是`offsetHeight`值，

滚动使用`scrollTo`元素自带的方法，传入的top值为offsetHeight值。

然后给该元素设置一个类名，让其高亮显示。

## 核心代码

```js
const UID = getUID()
function getNode(): HTMLElement {
    return document.querySelector(`#${UID}`)
}
```

```js
/**
 * @description 滚动到指定行
 * @param id dataSouce中的rowKey值
 * @returns
 */
function scrollToLine(id: string) {
    if (!id) return
    const node = getNode()
    const tableBody = node.querySelector('.ant-table-body')
    const resultTr: HTMLElement = tableBody.querySelector(
        `table tbody tr[data-row-key="${id}"]`,
    )
    const top = resultTr?.offsetTop || 0
    tableBody.scrollTo({ top, behavior: 'smooth' })
    resultTr.style.animationName = style.activeLine_animate
    resultTr.classList.add('activeLine')
    setTimeout(() => {
        resultTr.style.animationName = ''
        resultTr.classList.remove('activeLine')
    }, 2500)
}
```

使用：

```js
//滚动到指定行
function scrollTableToLine(id) {
  if (!id) return
  const parent = getParentNode(id, list.value, {
    fieldKey: 'baseAssetTypeId',
  })
  // 展开该父级
  expandedRowKeys.value = [...expandedRowKeys.value, parent.baseAssetTypeId]
  nextTick(() => {
    tableRef.value.scrollToLine(id)
  })
}
```

