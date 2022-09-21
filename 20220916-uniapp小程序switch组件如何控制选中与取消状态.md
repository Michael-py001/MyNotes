# 20220916-uniapp小程序switch组件如何控制选中与取消状态

![2022092021](https://pic.imgdb.cn/item/632a816816f2c2beb19dab76.gif)

## 存在的问题

```html
<switch :checked="power" @change="onSwitchOpen"  style="transform:scale(0.7)" color="#F9BA00"></switch>
```

以上的代码，如果想用变量`power`来控制是否选中只能在**初始状态**是正确的，当手动点击开关时，switch组件的状态不会被`power`的值给限制住，组件内部的状态和UI显示上还是会变动。而且只能通过`change`事件监听切换后的值。

## 解决方案

```html
<switch :checked="power" disabled  @click="onSwitchOpen"  style="transform:scale(0.7)" color="#F9BA00"></switch>
```

通过`disabled`属性把组件禁用，事件监听改成`click`，当用户点击时，把`power`的值取反即为切换后的值，提交给接口处理后，拿到最新的`power`值即可。这样就完全实现用`power`变量控制组件的显示。
