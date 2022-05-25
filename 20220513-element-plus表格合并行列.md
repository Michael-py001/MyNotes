# 20220513-element-plus表格合并行列

> 文档
>
> [Table 表格 | Element Plus (element-plus.org)](https://element-plus.org/zh-CN/component/table.html#合并行或列)

通过给 table 传入`span-method`方法可以实现合并行或列， 方法的参数是一个对象，里面包含：

当前行` row`、

当前列 `column`、

当前行号` rowIndex`、

当前列号 `columnIndex` 

四个属性。 该函数可以返回一个包含两个元素的数组，

第一个元素代表 `rowspan`，

第二个元素代表 `colspan`。 

也可以返回一个键名为` rowspan` 和` colspan` 的对象。

`rowspan`和`colspan`的含义根据下面的例子进行理解

## 官方案例

![image-20220513152722760](https://s2.loli.net/2022/05/13/njIQWzFUmVhkZ94.png)

返回数组形式, 返回`[1, 2]`，也就是`rowspan = 1 , colspan = 2`，含义：在这个格子，`rowspan `行数合并为`1行`，列数合并为`2行`.

```js
const arraySpanMethod = ({
  row,
  column,
  rowIndex,
  columnIndex,
}: SpanMethodProps) => {
  if (rowIndex % 2 === 0) {//当行号是双数时
    if (columnIndex === 0) {//当列号是首个时
      return [1, 2] //将两列合并成一列
    } else if (columnIndex === 1) {
      return [0, 0]//其余的不变
    }
  }
}
```

![image-20220513153445968](https://s2.loli.net/2022/05/13/5wmlfKORUiPJyY4.png)



```js
const objectSpanMethod = ({
  row,
  column,
  rowIndex,
  columnIndex,
}: SpanMethodProps) => {
  if (columnIndex === 0) {//当是首列时
    if (rowIndex % 2 === 0) {//如果行号是双数
      return {
        rowspan: 2, //合并成两行一列
        colspan: 1,
      }
    } else {
      return {
        rowspan: 0,
        colspan: 0,
      }
    }
  }
}
```

## 实战操作案例

在传给table的data数据中，加入自定义的属性：

```js
...
return
{
    ...s,
    rowspan: sk === 0 ? c.ad_group.length : 0,//这一项包含多少组数据，用来判断合并多少行
    index
}
```

```js
//合并数据
objectSpanMethod({
    row,
    column,
    rowIndex,
    columnIndex
}) {
    if (columnIndex === 0) {
        if (rowIndex === 0 || row.rowspan * 1 > 0) {
            return {
                rowspan: row.rowspan * 1,
                colspan: 1,
            }
        } else {
            return {
                rowspan: 0,
                colspan: 0,
            }
        }
    }
},
```

合并的效果

![image-20220513153809087](https://s2.loli.net/2022/05/13/x3JUwQS1YECWRlu.png)

案例2

```js
arraySpanMethod({
    row,
    column,
    rowIndex,
    columnIndex
}) {
    if (rowIndex === 0) {
        if (columnIndex === 0) {
            return [0, 0]
        } else if (columnIndex === 1) {
            return [0, 0]
        } else if (columnIndex === 2) {
            return [0, 0]
        } else if (columnIndex === 3) {
            return [1, 4]
        }
    }
},
```

![image-20220516134431748](https://s2.loli.net/2022/05/16/gYoVwDiTbMqfB1j.png)

```js
if (rowIndex === 0) {
    if (columnIndex === 0) {
        return [0, 0]
    } else if (columnIndex === 1) {
        return [0, 0]
    } else if (columnIndex === 2) {
        return [0, 0]
    } else if (columnIndex === 3) {
        return [1, 2]
    }
}
```

![image-20220516114539990](https://s2.loli.net/2022/05/16/UqRvHrDcGXpmiW8.png)

如果不处理：

![image-20220516135117122](https://s2.loli.net/2022/05/16/SFCxd4jiHG9rseh.png)