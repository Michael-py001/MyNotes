# 20240521-antd-vue Table组件合计行的实现

> 官方案例：
>
> https://www.antdv.com/components/table-cn#components-table-demo-sticky

顶部

![image-20240521153412839](https://s2.loli.net/2024/05/21/WzmRPaZntbK9Tc3.png)

底部

![image-20240521153436982](https://s2.loli.net/2024/05/21/Czrw5hZVHjnY3gc.png)

![image-20240521153838249](https://s2.loli.net/2024/05/21/Qybhf5zBT7uAi18.png)

实现过程：

使用TableSummary，TableSummaryRow，TableSummaryCell拼成完成一行。

TableSummaryCell设置index代表第几个格子。

也可以设置单元格合并`:col-span="6"`，对齐方式：`align="center"`