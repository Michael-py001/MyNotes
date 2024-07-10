# 20221219-Ant-design-vue-表格行列合并原理&多层级数据表格合并案例

> 使用的版本：Ant-design-vue 2.2.8
>
> 表头只支持列合并，使用 column 里的 colSpan 进行设置。 表格支持行/列合并，使用 render 里的单元格属性 colSpan 或者 rowSpan 设值为 0 时，设置的表格不会渲染。

表格合并主要涉及到column数据对象的customRender属性：

**customRender**：生成复杂数据的渲染函数，参数分别为当前行的值，当前行数据，行索引，@return 里面可以设置表格行/列合并,可参考 demo 表格行/列合并；[demo的在线预览地址](https://codesandbox.io/s/biao-ge-xing-lie-he-bing-ant-design-vue-next-forked-wpnzxd?file=/src/Demo.vue)

## **customRender的参数**：

`customRender: ({ text,record, index }) =>{}` 

- text 当前行的值
- record 当前行的数据
- index 当前行索引

## customRender的返回值：

这个函数需要返回一个对象：

```js
return obj = {
    children: text, //显示的值
    props: {
        colSpan:xxx, //该格子的列格子数 
        rowSpan:xxx  //该格子的行格子数
    },
};
```

- props.colSpan  该格子的列格子数  (从左往右)
- props.rowSpan 该格子的行格子数（从上往下）

`props.colSpan`的案例demo：

![image-20221219110131990](https://f.pz.al/pzal/2022/12/19/d3c5f8ca0aa5f.png)

`props.rowSpan`的案例demo：

![image-20221219112047170](https://f.pz.al/pzal/2022/12/19/3b756aa216598.png)

### 注意：

如果设置了`props.colSpan=2` ，这个格子水平方向：列占格数变成了两列，会把右边的格子往右边挤，导致最后一格格子会超出表格，表现为：表格多了一列。
![image-20221219113509631](https://f.pz.al/pzal/2022/12/19/7ae56d2152a73.png)

如果设置了`props.rowSpan=2` ，这个格子的垂直方向：行占格数变成了2格，霸占了下一行的格子，导致下一行的原本位置的格子会往右移动，最后一个格子会超出表格，导致多了一列：
![image-20221219112849714](https://f.pz.al/pzal/2022/12/19/280f5474a28ed.png)

demo中的座机号码需要合并的话，需要设置两行的rowSpan，因为其中一行扩大占行数，肯定会影响下一行的表格显示，此时需要把下一行的格子的占行数设置成0，即不显示了：

![image-20221219114732338](https://f.pz.al/pzal/2022/12/19/fc33443f292f8.png)

最后一行的分析：把`renderContent`函数设置到第二行开始的所有列，就会把每一列的index=4(第五行)的列占格子数设置成0列：`props.colSpan = 0`，这样就只有第一列的name列渲染，其他的都不显示。

```js
const renderContent = ({ text, index }) => {
  const obj = {
    children: text,
    props: {},
  };

  if (index === 4) {
    obj.props.colSpan = 0;
  }

  return obj;
};
```

# 案例

> [ant-design-vue动态表格合并案例_vue.js_脚本之家 (jb51.net)](https://www.jb51.net/article/259578.htm)

## 一

![image-20230411172149074](https://s2.loli.net/2023/04/11/YT6vAg4N8mIhuV2.png)

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

用法：

```js
const customRenderList = ['actType', 'organizationName', 'executiveLevel', 'sysOrganizationId']; //需要合并的字段
tableData.value = mergeCell(data.records, customRenderList);
```



## 多单元格合并案例

![image-20230411172437643](https://s2.loli.net/2023/04/11/9SPDmendgtYq4zv.png)

![image-20230411172451836](https://s2.loli.net/2023/04/11/rq4a2YGfKiJmy7p.png)

```js
/**
     * 
     * @param {String} key 需要查找的key字段值
     */
    //  合并单元格
    const combineRow = (tableData,key)=>{
      try {
        for (let i = 0; i < tableData.length; i++) {
          const item = tableData[i]
          let count = 1
          for (let j = i + 1; j < tableData.length; j++) {
            let nextItem = tableData[j]
            // 下一项跟当前项的columnName相同时
            if(item[key] === nextItem[key]) {
              count ++
              // count会每一轮循环增加，合并单元也跟着增加
              item[`${key}RowSpan`] = count
              // 往后相同的值都设为空单元格
              tableData[j][`${key}RowSpan`] = 0
              // 特殊处理
              if(item.assetGroupId==="1600008673084002362") {
                item[`${key}RowSpan`] = 1
              }
              // 到达最后一个结束
              if (j === tableData.length - 1) {
                  return
              }
            }
            else {
              // 指针跳转到下一个，从下一排开始, i回到初始位置
              i = j - 1
              count = 1
              break
            }
          }
          
        }
      } catch (error) {
        console.log("error:",error)
      }
      
    }
```

合并用法：传入数据和需要合并的字段name

```js
combineRow(tableData.value,'level0Name')
combineRow(tableData.value,'level1Name')
combineRow(tableData.value,'level2Name')
combineRow(tableData.value,'level3Name')
```

此方法为提前计算好合并单元格的数量：

![image-20240527114043605](https://s2.loli.net/2024/05/27/JPYzwQKTufslcyI.png)

### 优化版

此方法为在column中计算时才进行判断合并，减少一层循环，可读性更高，更简单。

```js
{
  title: '一级分类',
    dataIndex: 'level0Name',
      customRender: ({ text, record, index }) => {
        return {
          children: `${text ? text : '/'}`,
          props: {
            rowSpan: combineRowPro(text, 'level0Name', tableData.value, index),
          },
        }
      },
},
```

核心：

- 如果当前行和上一行的值相同，则合并单元格为0
- 如果当前行和下一行的值相同，则合并单元格+1，否则中断

```js
function combineRowPro(text, key, tableData, index) {
  // 如果当前行和上一行的值相同，则合并单元格为0
  if (index !== 0 && text === tableData[index - 1]?.[key]) {
    return 0
  }
  let rowSpan = 1
  // 如果当前行和下一行的值相同，则合并单元格+1，否则中断
  for (let i = index + 1; i < tableData.length; i++) {
    if (text === tableData[i][key]) {
      rowSpan++
    } else {
      break
    }
  }
  return rowSpan
}
```

