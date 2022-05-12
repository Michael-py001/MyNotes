# 20220511-输入框placeholder文字滚动案例

> [PC 官网 - 首页 (fusion.design)](https://fusion.design/pc/doc/design/布局/16?themeid=2)

![scroll](https://s2.loli.net/2022/05/11/KcduLwHkaRsnvWt.gif)

html:

```html
<div class="dynamic-list">
    <ul style="margin-top: 0px;">
        <li>输入关键词直达组件文档</li>
        <li>输入关键词搜索站点、物料</li>
        <li>输入关键词搜索帮助文档</li>
    </ul>
</div>
```

![image-20220511173950454](https://s2.loli.net/2022/05/11/uxySzkc3leTPKwm.png)

css:

```css
.dynamic-list {
    overflow: hidden;
    background-color: inherit;
    padding-left: 20px;
    z-index: 1;
    pointer-events: none;
    position: absolute;
    top: 0;
    bottom: 0;
    left: 8px;
    right: 0;
}
```

![image-20220511174130629](https://s2.loli.net/2022/05/11/XfbcE6Sm5puJ8UF.png)

实现的思路：

`dynamic-list`使用绝对定位在input上面，z-index层级比input框高，当点击改区域时，自身消失，下面的input框显示，并且设置聚焦。当`input`失焦时，`dynamic-list`再显示。

滚动的实现思路：

`ul`列表使用`margin-top`一直减小到显示下一组文字，当结束滚动时，立刻计算文字的数组排列顺序，最顶部的放到最后面。继续下一轮的滚动，再重置顺序，如此反复实现无限滚动。



