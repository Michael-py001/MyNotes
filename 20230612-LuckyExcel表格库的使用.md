# 20230612-LuckyExcel表格库的使用

> [Luckysheet/README-zh.md at master · dream-num/Luckysheet · GitHub](https://github.com/dream-num/Luckysheet/blob/master/README-zh.md)
>
> [Luckyexcel/README-zh.md at master · dream-num/Luckyexcel · GitHub](https://github.com/dream-num/Luckyexcel/blob/master/README-zh.md)

- Luckysheet ，一款纯前端类似excel的在线表格，功能强大、配置简单、完全开源。

- Luckyexcel，是一个适配 [Luckysheet](https://github.com/mengshukeji/Luckysheet) 的excel导入导出库，只支持.xlsx格式文件（不支持.xls）。



## 使用案例

![image-20230616115003018](https://s2.loli.net/2023/06/16/MNQY8sR4t75JCAS.png)

```html
<script src="https://cdn.jsdelivr.net/npm/luckyexcel/dist/luckyexcel.umd.js"></script>
<script>
    // 先确保获取到了xlsx文件file，再使用全局方法window.LuckyExcel转化
    LuckyExcel.transformExcelToLucky(
        file, 
        'xxxname',//名称
        function(exportJson, luckysheetfile){
            // 获得转化后的表格数据后，使用luckysheet初始化，或者更新已有的luckysheet工作簿
            // 注：luckysheet需要引入依赖包和初始化表格容器才可以使用
            luckysheet.create({
                container: 'luckysheet', // luckysheet is the container id
                data:exportJson.sheets,
                title:exportJson.info.name,
                userInfo:exportJson.info.name.creator
            });
        },
        function(err){
            logger.error('Import failed. Is your fail a valid xlsx?');
        }
    );
</script>
```

> 详细使用案例：[Luckyexcel/index.html at master · dream-num/Luckyexcel · GitHub](https://github.com/dream-num/Luckyexcel/blob/master/src/index.html)

## 实际使用案例

```html
<template>
    <div>
        <link rel="stylesheet" href="/luckysheet/pluginsCss.css" />
        <link rel="stylesheet" href="/luckysheet/plugins.css" />
        <link rel="stylesheet" href="/luckysheet/luckysheet.css" />
        <link rel="stylesheet" href="/luckysheet/iconfont.css" />

        <SearchForm @search="onSearch" @reset="onReset"/>

        <a-row type="flex" justify="end">
            <a-button type="primary" @click="exportGraph">导出</a-button>
        </a-row>
        <a-spin :spinning="loading">
            <div style="min-height: 500px;">
                <div class="luckysheet-container" 
                    :style=" !resource ? 'height: 0; overflow: hidden;' : ''" 
                    id="luckysheet">
                </div>
            </div>
        </a-spin>
    </div>
</template>
```

```js

onMounted(() => {
    const script1 = document.createElement('script');
    script1.src = '/luckysheet/plugin.js';
    document.body.appendChild(script1);

    const script2 = document.createElement('script');
    script2.src = '/luckysheet/luckysheet.umd.js';
    document.body.appendChild(script2);
});
```

```js
const fetchStatisGraph = async (data) => {
    const res = await apis.statisGraph.exportTradeReport(data);
    resource.value = res;

    const blob = new Blob([res.data], { type: "application/vnd.ms-excel;charset=utf8" });
    const stream = window.URL.createObjectURL(blob);
    

    LuckyExcel.transformExcelToLuckyByUrl(stream, '报表', function (exportJson, luckysheetfile) {
        if (exportJson.sheets == null || exportJson.sheets.length == 0) {
            alert('Failed to read the content of the excel file, currently does not support xls files!')
            return
        }
        console.log('exportJson', exportJson)
        jsonData.value = exportJson

        window.luckysheet.destroy();

        window.luckysheet.create({
            container: 'luckysheet', // luckysheet is the container id
            showinfobar: false,
            data: exportJson.sheets,
            title: exportJson.info.name,
            userInfo: exportJson.info.name.creator,
        });

        loading.value = false;
    });
}
```

