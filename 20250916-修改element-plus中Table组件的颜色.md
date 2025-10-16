# 20250916-修改element-plus中Table组件的颜色

> 参考文章：https://blog.csdn.net/Turn_to_empty/article/details/136675533

## element-plus 中的css变量

```css
.el-table {
    // 表格边框的颜色，可以通过这个变量来设置表格的边框颜色。
    --el-table-border-color: var(--el-border-color-lighter);
    // 表格边框的样式，一般为实线或虚线，可以通过这个变量来设置表格的边框样式。
    --el-table-border: 1px solid var(--el-table-border-color);
    // 表格中文字的颜色，可以通过这个变量来设置表格中文字的颜色。
    --el-table-text-color: var(--el-text-color-regular);
    // 表头中文字的颜色，可以通过这个变量来设置表头中文字的颜色。
    --el-table-header-text-color: var(--el-text-color-secondary);
    // 鼠标悬停在行上时的背景色，可以通过这个变量来设置鼠标悬停时行的背景色。
    --el-table-row-hover-bg-color: var(--el-fill-color-light);
    // 当前行的背景色，可以通过这个变量来设置当前行的背景色。
    --el-table-current-row-bg-color: var(--el-fill-color-light);
    // 表头的背景色，可以通过这个变量来设置表头的背景色。
    --el-table-header-bg-color: var(--el-bg-color);
    // 固定列的阴影样式，可以通过这个变量来设置固定列的阴影样式。
    --el-table-fixed-box-shadow: var(--el-box-shadow-light);
    // 表格的背景色，可以通过这个变量来设置表格的背景色。
    --el-table-bg-color: var(--el-fill-color-blank);
    // 表格行的背景色，可以通过这个变量来设置表格行的背景色。
    --el-table-tr-bg-color: var(--el-fill-color-blank);
    // 展开行的背景色，可以通过这个变量来设置展开行的背景色。
    --el-table-expanded-cell-bg-color: var(--el-fill-color-blank);
    // 左侧固定列的阴影样式，可以通过这个变量来设置左侧固定列的阴影样式。
    --el-table-fixed-left-column: inset 10px 0 10px -10px rgba(0, 0, 0, 0.15);
    // 右侧固定列的阴影样式，可以通过这个变量来设置右侧固定列的阴影样式。
    --el-table-fixed-right-column: inset -10px 0 10px -10px rgba(0, 0, 0, 0.15);
}
```



## 修改Table tr的鼠标悬浮颜色

从上面的css可以看到：`--el-table-row-hover-bg-color: var(--el-fill-color-light);`，这个最终是使用了`--el-fill-color-light`这个颜色，所以我们只需要覆盖重写这个css变量就行：

```css
root: {
	 --el-fill-color-light: red;
}
```

![image-20250916111438139](https://s2.loli.net/2025/09/16/wlZJfxyXe4TgHbp.png)