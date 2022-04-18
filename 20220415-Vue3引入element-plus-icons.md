# 20220415-Vue3引入element-plus-icons

## 安装

```bash
# NPM
$ npm install @element-plus/icons-vue
# Yarn
$ yarn add @element-plus/icons-vue
# pnpm
$ pnpm install @element-plus/icons-vue
```

## 引入

```js
//element-plus-icons.js
import * as ElIcons from '@element-plus/icons'

export default (app) => {
  for (const name in ElIcons){
    app.component(name,ElIcons[name])
  }
}
```

```js
//main.js  引入element plus icons
import installElementPlusIcons from '@/plugins/element-plus-icons'
installElementPlusIcons(app)
```

Vue3引入组件

> [Component Registration | Vue.js (vuejs.org)](https://vuejs.org/guide/components/registration.html#global-registration)

## 使用中出现的问题：svg大小不会改变

使用官网的用法，svg的大小不会改变，

```html
<el-icon color="#fff" size="30" v-if="closeTab">
    <ArrowRightBold />
</el-icon>
```

![image-20220415104845648](https://s2.loli.net/2022/04/15/BzDlAFSsuk7aWcH.png)

经过跟官网的样式对比后，发现了svg的width,height设置的值是1em，而em单位是根据这个元素的font-size决定的，而font-size的值是继承自`el-icon`的属性

![image-20220415110921806](https://s2.loli.net/2022/04/15/pBPNXs3YI8mWSwy.png)

到这里就发现了是上面 `* {font-size:14px}`影响到了继承属性，这是在全局`reset.css`里面写的，所以以后不能在全局写默认字体大小，这可能会影响到一些库的继承属性。



单独用svg的用法

```html
<ArrowRightBold style="width: 1.5em; height: 1.5em;" v-if="closeTab"/>
<ArrowLeftBold style="width: 1.5em; height: 1.5em;" v-else/>
```

