## 20211018-自定义导航苹果机position: sticky不生效解决方案



方案： 使用fixed定位，这样会导致导航栏浮动，下面的内容会自动顶到最上面，所以导航栏下面部分使用`margin-top`撑起，值为整个自定义导航栏的高度。

```html
<TopColumn></TopColumn>
<view class="Index2-content" :style="{'margin-top':holdHeightLess+'px'}"><view>
```

![image-20211021111736828](https://i.loli.net/2021/10/21/IV1bDWU2t7ymAP6.png)

![image-20211021111837273](https://i.loli.net/2021/10/21/hdU8WMaEwqAJki3.png)