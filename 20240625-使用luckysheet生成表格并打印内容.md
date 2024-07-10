# 20240625-使用luckysheet生成表格并打印内容

> 官方文档：
>
> https://dream-num.github.io/LuckysheetDocs/zh/guide/



## 获取数据

接口数据需要返回流文件，前端设置responseType: blob

```js
const res = await api(restQuery)

if (res.data.size < 250) {
  window.luckysheet?.destroy()
  resource.value = 'empty'
  loading.value = false
  return
}

resource.value = res
const blob = new Blob([res.data], { type: 'application/vnd.ms-excel;charset=utf8' })
const stream = window.URL.createObjectURL(blob)
LuckyExcel.transformExcelToLuckyByUrl(
  stream,
  '报表',
  function (exportJson, luckysheetfile) {
    if (exportJson.sheets == null || exportJson.sheets.length == 0) {
      alert(
        'Failed to read the content of the excel file, currently does not support xls files!',
      )
      return
    }
    jsonData.value = exportJson

    window.luckysheet?.destroy()

    window.luckysheet.create({
      container: 'luckysheet', // luckysheet is the container id
      showinfobar: false,
      data: exportJson.sheets,
      title: exportJson.info.name,
      userInfo: exportJson.info.name.creator,
    })
    print()
    loading.value = false
  },
)
```

## 获取选区内容，并截取成图片打印

```js

// 获取表格中包含内容的row，column
function getExcelRowColumn() {
  const sheetData = window.luckysheet.getSheetData()
  let objRowColumn = {
    row: [null, null], //行
    column: [null, null], //列
  }
  sheetData.forEach((item, index) => {
    //行数
    item.forEach((it, itemIndex) => {
      if (it !== null) {
        if (objRowColumn.row[0] == null) objRowColumn.row[0] = index // row第一位
        objRowColumn.row[1] = index //row第二位
        if (objRowColumn.column[0] == null) objRowColumn.column[0] = itemIndex //column第一位
        objRowColumn.column[1] = itemIndex //column第二位
      }
    })
  })
  return objRowColumn
}

function printExcel() {
  let RowColumn = getExcelRowColumn() // 获取有值的行和列
  RowColumn.column[0] = 0 //因需要打印左边的边框，需重新设置
  window.luckysheet.setRangeShow(RowColumn) // 进行选区操作

  setTimeout(() => {
    let src = window.luckysheet.getScreenshot() // 生成base64图片
    let $img = `<img src=${src} style="width: 100%" />`
    document.querySelector('.print-area').innerHTML = $img
    setTimeout(() => {
      window.onafterprint = function () {
        window.close()
      }
      window.print()
    }, 1000)
  }, 1000)
}
```

渲染展示表格：

![image-20240625172857690](https://s2.loli.net/2024/06/25/LnMdSmUV8Awj2s4.png)

打印时展示图片:

![image-20240625172918611](https://s2.loli.net/2024/06/25/LDPk65ZgWmAS9KG.png)

这样做的目的是为了让表格所有列都展示出来，打印时可以完整打印。

### 打印样式

```html
<div v-show="resource !== 'empty'" style="min-height: 500px">
  <div
       id="luckysheet"
       class="luckysheet-container"
       :style="!resource ? 'height: 0; overflow: hidden;' : ''"
       ></div>
  <div class="print-area"></div>
</div>
```



```less
*,
*:before,
*:after {
  box-sizing: inherit;
  color: inherit;
}
html,
body {
  margin: 0;
}
body {
  background-color: #333333;
  font-family: sans-serif;
  max-width: 80em;
  margin-left: auto;
  margin-right: auto;
}
.ant-empty-description {
  color: #fff !important;
}
@page {
  size: A4;
  margin: 0;
}
.luckysheet-container,
.print-area {
  position: relative;

  display: grid;
  grid-auto-rows: max-content;
  grid-row-gap: 16px;

  border: 1px #d3d3d3 solid;
  border-radius: 5px;
  // padding: 0.5in;
  background-color: white;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  margin: 8px auto;
  height: 297mm;
  width: 210mm;
}
.print-area {
  padding: 0.5in;
}
@media print {
  body {
    background-color: white !important;
  }
  thead {
    display: table-row-group;
  }
  .luckysheet-container,
  .print-area {
    margin: 0px;
    border: none;
    border-radius: initial;
    box-shadow: none;
    background-color: inherit;
    page-break-before: always;
  }
  .luckysheet-container,
  .btns {
    display: none;
  }
  .print-area {
    display: block;
  }
}
```

