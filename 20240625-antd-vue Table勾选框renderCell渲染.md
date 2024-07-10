# 20240625-antd-vue Table勾选框renderCell渲染

场景: 设置了勾选框的table，把不想要勾选的列隐藏勾选框。

![image-20240625112617839](https://s2.loli.net/2024/06/25/9zTtV3uBiIkX4hl.png)

```vue
<template>
  <a-table
    :dataSource="dataSource"
    :columns="columns"
    :rowSelection="rowSelection"
  />
</template>

<script>
export default {
  data() {
    return {
      dataSource: [], // 你的数据源
      columns: [], // 你的列定义
      rowSelection: {
        renderCell: (checked, record, index, originNode) => {
          // 在这里，你可以根据你的行合并逻辑来决定是否渲染选择框
          // 例如，你可以检查当前行是否是合并的第一行，如果是，那么就渲染选择框，否则就不渲染
          if (/* 这是合并的第一行 */) {
            return originNode;
          }
          return null;
        },
      },
    };
  },
};
</script>
```

实现效果：

![image-20240625163309146](https://s2.loli.net/2024/06/25/rlytnod6UqTBQVw.png)

```js
renderCell: (checked, record, index, originNode) => {
  if (['期初余额', '合计'].includes(record._text)) {
    return null
  }
  return originNode
},
```

