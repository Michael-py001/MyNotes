# 20230112-antd Vue Table的customCell使用

> [记录Ant-Vue-table中customCell改变单元格样式_凉辞_的博客-CSDN博客_customcell](https://blog.csdn.net/qq_38344500/article/details/109220830?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2~default~CTRLIST~Rate-1-109220830-blog-109238602.pc_relevant_multi_platform_whitelistv3&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2~default~CTRLIST~Rate-1-109220830-blog-109238602.pc_relevant_multi_platform_whitelistv3&utm_relevant_index=1)
>
> [Antd-vue中table组件的customCell用法_S_Yueee的博客-CSDN博客_customcell vue](https://blog.csdn.net/qq_33262275/article/details/109238602?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.control&dist_request_id=6b355da2-8d9a-4124-bdd2-3b9d5e81816b&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.control)



#### customRow 用法 [#](https://www.antdv.com/components/table-cn/#customRow-用法)

适用于 `customRow` `customHeaderRow` `customCell` `customHeaderCell`。遵循[Vue jsx](https://github.com/vuejs/babel-plugin-transform-vue-jsx)语法。

```jsx

<Table
  customRow={(record) => {
    return {
      xxx... //属性
      onClick: (event) => {},       // 点击行
      onDblclick: (event) => {},
      onContextmenu: (event) => {},
      onMouseenter: (event) => {},  // 鼠标移入行
      onMouseleave: (event) => {}
    };
  }}
  customHeaderRow={(columns, index) => {
    return {
      onClick: () => {},        // 点击表头行
    };
  }}
/>
```

![image-20230113103606597](https://s2.loli.net/2023/01/13/y1cg7qvV2SWCfTo.png)

设置单元格的属性，意味着可以设置样式，添加事件监听等操作；

## 设置单元格样式

```js
const columns = [
...
{ 
  title: '时间(s)',
  dataIndex: 'time', 
  key:'time',
  align: "center", 
  width: 100, 
  customCell:this.renderTimeBackground
},
...
]
```

在`methods`中定义`renderTdBackground()`和`renderTimeBackground()`方法

```js
renderTimeBackground(record){
	return this.renderTdBackground(record,1)
},
```

当时间超时`timeOver == 1`的时候，说明`time`超时了，需要标红这个单元格

```js
renderTdBackground(record,flag){
  if(flag == 1){
    if(record.timeOver == 1){
      return {
        style: {
          'background-color': 'rgb(255,150,150)',
        },
      }
    }
  }
},
```

