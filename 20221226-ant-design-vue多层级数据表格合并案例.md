# 20221226-ant-design-vue多层级数据表格合并案例

> [ant-design-vue动态表格合并案例_vue.js_脚本之家 (jb51.net)](https://www.jb51.net/article/259578.htm)

```js
function mergeCell(data, keys) {
    let tempValue = '';
    let tempId = '';
    for (let kIndex = 0; kIndex < keys.length; kIndex++) {
      const key = keys[kIndex];
      for (let i = 0; i < data.length; ) {
        let rowSpan = 0;
        if (tempValue[key] !== data[i][key]) {
          tempValue = data[i][key];
          tempId = data[i]['sysOrganizationId'];
          for (let j = i; j < data.length; j++) {
            const item = data[j];
            if (
              item[key] === tempValue &&
              item['sysOrganizationId'] === tempId &&
              item['executiveLevel'] === data[i]['executiveLevel']
            ) {
              rowSpan += 1;
            } else {
              break;
            }
          }
        }
        data[i][`${key}RowSpan`] = rowSpan;
        for (let j = i + 1; j < i + rowSpan && j < data.length; j++) {
          data[j][`${key}RowSpan`] = 0;
        }
        i += rowSpan;
      }
    }
    return data;
  }
```

