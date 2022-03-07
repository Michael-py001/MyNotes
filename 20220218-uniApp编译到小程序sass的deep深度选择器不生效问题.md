# 20220218-uniApp编译到小程序sass的deep深度选择器不生效问题

## 情景

1. 修改uView组件上的样式，把按钮的绝对定位去掉，文字一行显示，但是不生效

```js
.u-btn-text {
   white-space: nowrap;
   position: static !important;
   transform:translate(0) !important;
 }
```

按道理，使用`/deep/`是应该生效了，但是小程序上不生效。



## 解决方案

只能在全局覆盖这个class的样式。

```css
//app.
.u-btn-text {
   white-space: nowrap;
   position: static !important;
   transform:translate(0) !important;
 }
```

在`App.vue`中 加入上述样式就能生效了

![image-20220218151320290](https://s2.loli.net/2022/02/18/AZau8hbR5dyoUGJ.png)

![image-20220218151146110](https://s2.loli.net/2022/02/18/AMj1ks3IqiKorFT.png)

