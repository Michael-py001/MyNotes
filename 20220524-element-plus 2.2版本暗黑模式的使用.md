# 20220524-element-plus 2.2版本暗黑模式的使用

> 官方文档
>
> [暗黑模式 | Element Plus (element-plus.org)](https://element-plus.org/zh-CN/guide/dark-mode.html#如何启用？)

## 颜色切换的原理

element ui内置了一套暗黑模式的css变量，激活这套样式的方法就是在html标签中添加`dark`类名， 因为这套css变量是写在`html.dark`下的。

## 如何启用？

下面是官方文档的介绍：

首先你可以创建一个开关来控制 `暗黑模式` 的 class 类名。

> 如果您只需要暗色模式，只需在 html 上添加一个名为 `dark` 的类 。

```html
<html class="dark">
  <head></head>
  <body></body>
</html>
```

> 如果您想动态切换，建议使用 [useDark | VueUse](https://vueuse.org/core/useDark/)。

只需要如下在项目入口文件修改一行代码：

```js
// main.ts
// 如果只想导入css变量
import 'element-plus/theme-chalk/dark/css-vars.css'
```

此时，element的组件在激活暗黑模式时，应该都是暗黑色了。但是我们自己的组件并没有变色，所以我们需要给自己的组件样式也添加暗黑色。

## 切换模式

切换开关

```js
switchTheme(value) {
    this.$SetStorage('dark',value?1:0)
    document.documentElement.classList.toggle('dark')
},
```

`App.vue`在挂载时从本地存储中取出设置了的模式重新设置，不然不会记忆上次切换的模式

```js
	mounted() {
        let isDark = this.$GetStorage('dark')
        if(isDark) {
            document.documentElement.classList.add('dark')
        }
        else {
            document.documentElement.classList.remove('dark')
        }
    }
```



## 给组件添加暗黑css

### 方法一：使用css变量

在`html.dark`下定义自己的css变量，各个组件都使用这些变量

### 方法二：强制覆盖样式

> 如果一开始项目没有做好规划统一使用css变量，后期修改每个组件的样式的十分繁琐的，这里使用覆盖样式的方案解决。

新建一个`dark.scss`，

```scss
$dark-bg-color:#181818;
$dark-bg-color-text:#989696;
html.dark {
  /* 自定义深色背景颜色 */
  .el-table tr {
    background-color:#141414
  }
  .el-table__body-wrapper .el-table__body .el-table__row:hover td {
    background-color:#202020 !important
  }
}
```

## 注意事项

根据`element-plus`的文档，需要引入暗黑css样式

```js
//main.js
import 'element-plus/theme-chalk/dark/css-vars.css'
```

但是，如果在main.js同时也引入自己的`daek.scss`，开发环境运行是没有问题的，但是打包好后，element-plus的样式会丢失，很可能是由于`tree-shaking`的原因给摇丢了。

经过一番探索，发现在`App.vue`中引入自己的样式，打包后element的样式不会丢失，而是正常一起共存。

## 效果

![image-20220524155248498](https://s2.loli.net/2022/05/24/KXt5sQrMvq7eG8i.png)
