# 20240629-Antd-vue实现树形Table展开连接线

> 参考文章：https://blog.csdn.net/qq_43760344/article/details/139832727

![image-20240629151850580](https://s2.loli.net/2024/06/29/4365QICoRJlgsWj.png)

![image-20240629151901282](https://s2.loli.net/2024/06/29/FV5CXcu8BPM1aHR.png)

![image-20240704175329130](https://s2.loli.net/2024/07/04/YLHm3FqsG6Mkph2.png)

## 实现思路

实现思路是在Table组件提供的`expandIcon`属性入手，传入一个自己的Icon，并且是加上了连接线的Icon。

### T型连接线

以下是在原本的展开icon旁边加上一个div，这是第一种链接类型：T型。

如果是同级中不是最后一个节点时，显示T型

![image-20240704175446859](https://s2.loli.net/2024/07/04/7Z4cP85lSQ9XGC3.png)

![image-20240704175950420](https://s2.loli.net/2024/07/04/PLi7gEtO8s3ov2z.png)

### 竖线：底部连接线

竖线外层高度是100%，里面的div高度取一半，达到显示底部连接线的效果。

如果是同级中不是最后一个节点时，显示竖线



![image-20240704180154071](https://s2.loli.net/2024/07/04/pPwFNbyCv3lAIcx.png)![image-20240704180857022](https://s2.loli.net/2024/07/04/u5HKnWoacMlLhVS.png)

![image-20240704180925107](https://s2.loli.net/2024/07/04/CVSMByl5PXud81x.png)

### L型

当是同一级中最后一个节点时，显示L型连接线

![image-20240704181236531](https://s2.loli.net/2024/07/04/tELQfbzWKqBMlVa.png)

### 左侧竖线

左侧竖线是会存在多条的，因为有多个父级，每个层级都需要展示一条当前位置的竖线.

![image-20240704182220032](C:\Users\WuShiLi\AppData\Roaming\Typora\typora-user-images\image-20240704182220032.png)

![image-20240710180729784](https://s2.loli.net/2024/07/10/N3lEQAzma4L6epg.png)

```jsx
let prefix = null
// 找到父节点，如果父节点是最后一个节点就不需要显示前边的竖线
const parentPath = findPathToNode(tableData.value, record[tableRowKey])
// parentPath是当前节点和他的所有父级组成的数组，结构为：[parentLevel1,parentLevel2,...,currentNode]
prefix = showPrefix(parentPath.slice(1)) // showPrefix会遍历这个数组，返回多个连接线节点，比如：[|,|,L]或者[|,|,├]
return (
  <>
  {prefix}
{expanded ? record.children && fixedExpandLine(record?.nodeLevel) : null}
{record.someItemHasChild ? paddingDiv : null}
{expanded ? expandedIcon : unExpandedIcon}
</>
)
```



```jsx
// 非叶子节点的T形结构
const showPrefix = parentPath => {
  return parentPath.map((item, index) => {
    // 
    if (index === parentPath.length - 1) {
      // 当前节点根据节点类型显示不同的连接线：最后一个节点显示L形，非最后一个节点├形
      if (item?.nodeType === 1) {
        return fixedT(item?.nodeLevel, item)
      } else if (item?.nodeType === 2) {
        return fixedL(item?.nodeLevel, item)
      }
    } else {
      // 当父节点（包括祖先节点）不是最后一个节点时该行前才会显示竖线
      if (item?.nodeType === 1) {
        return (
          <div
            style={{
              display: 'flex',
                alignItems: 'center',
                  height: '100%',
                    top: 0,
                      position: 'absolute',
                        left:
                        (item?.nodeLevel - 1) * indentSize +
                          disOffset -
                          thickness / 2 +
                          'px',
            }}
            >
            <div
              style={{
                height: '100%',
                  borderLeft: borderStyle,
              }}
              ></div>
          </div>
        )
      }
    }
  })
}
```



### 节点总结

一共最多是三个元素：T/L连接线 、I 竖型底部连接线、展开icon 



![image-20240710181754195](https://s2.loli.net/2024/07/10/TpBLgRGtrqkuneS.png)