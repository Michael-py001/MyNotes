# VueRouter配合使用elementUI

## 安装

安装elementUI

```
npm i element-ui -S
```

## 引入

### 引入 Element

你可以引入整个 Element，或是根据需要仅引入部分组件。我们先介绍如何引入完整的 Element。

#### [#](https://element.eleme.cn/#/zh-CN/component/quickstart#wan-zheng-yin-ru)完整引入

在 main.js 中写入以下内容：

```javascript
import Vue from 'vue';
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
import App from './App.vue';

Vue.use(ElementUI);

new Vue({
  el: '#app',
  render: h => h(App)
});
```

以上代码便完成了 Element 的引入。需要注意的是，样式文件需要单独引入。

#### [#](https://element.eleme.cn/#/zh-CN/component/quickstart#an-xu-yin-ru)按需引入

借助 [babel-plugin-component](https://github.com/QingWei-Li/babel-plugin-component)，我们可以只引入需要的组件，以达到减小项目体积的目的。

首先，安装 babel-plugin-component：

```bash
npm install babel-plugin-component -D
```

然后，将 .babelrc 修改为：

```json
{
  "presets": [["es2015", { "modules": false }]],
  "plugins": [
    [
      "component",
      {
        "libraryName": "element-ui",
        "styleLibraryName": "theme-chalk"
      }
    ]
  ]
}
```

接下来，如果你只希望引入部分组件，比如 Button 和 Select，那么需要在 main.js 中写入以下内容：

```javascript
import Vue from 'vue';
import { Button, Select } from 'element-ui';
import App from './App.vue';

Vue.component(Button.name, Button);
Vue.component(Select.name, Select);
/* 或写为
 * Vue.use(Button)
 * Vue.use(Select)
 */

new Vue({
  el: '#app',
  render: h => h(App)
});
```

完整组件列表和引入方式（完整组件列表以 [components.json](https://github.com/ElemeFE/element/blob/master/components.json) 为准），请查看:https://element.eleme.cn/#/zh-CN/component/quickstart

本案例中使用全局引入。

## 引入Container布局容器

> 官方链接：https://element.eleme.cn/#/zh-CN/component/container

在`App.vue`主入口插入以下代码：

​	在main里插入`router-view`

```html
 <el-container>
   <el-header>Header</el-header>
   <el-main>
     <!-- 在main里插入router-view -->
     <router-view />
   </el-main>
   <el-footer>Footer</el-footer>
</el-container>
```

## 引入NavMenu 导航菜单

在`<el-header>`中插入组件：

```html
  <el-container>
      <el-header>
        <!-- 在herder中插入navMenue -->
        <!-- 
          default-active--当前激活菜单的 index
          mode--模式--horizontal / vertical
         -->
        <el-menu
          :default-active="activeIndex2"
          class="el-menu-demo"
          mode="horizontal"
          @select="handleSelect"
          background-color="#545c64"
          text-color="#fff"
          active-text-color="#ffd04b"
        >
          <el-menu-item index="1">个人中心</el-menu-item>
          <el-submenu index="2">
            <template slot="title">我的工作台</template>
            <el-menu-item index="2-1">选项1</el-menu-item>
            <el-menu-item index="2-2">选项2</el-menu-item>
            <el-menu-item index="2-3">选项3</el-menu-item>
            <el-submenu index="2-4">
              <template slot="title">选项4</template>
              <el-menu-item index="2-4-1">选项1</el-menu-item>
              <el-menu-item index="2-4-2">选项2</el-menu-item>
              <el-menu-item index="2-4-3">选项3</el-menu-item>
            </el-submenu>
          </el-submenu>
          <el-menu-item index="3" disabled>消息中心</el-menu-item>
          <el-menu-item index="4"
            ><a href="https://www.ele.me" target="_blank"
              >订单管理</a
            ></el-menu-item
          >
        </el-menu>
      </el-header>
      <el-container>
        <el-aside width="200px">Aside</el-aside>
        <el-container>
          <el-main>
            <!-- 在main里插入router-view -->
            <router-view />
          </el-main>
          <el-footer>Footer</el-footer>
        </el-container>
      </el-container>
    </el-container>
```

在js中的data设置`activeIndex`

```js
data: function () {
    return {
      activeIndex: "1",
    };
  },
```

## 引入侧边栏aside bar

```html
<el-aside width="160px">
          <!-- :span 控制宽度 -->
          <el-col :span="24">
          <h5>自定义颜色</h5>
          <el-menu
            default-active="2"
            @open="handleOpen"
            @close="handleClose"
            background-color="#545c64"
            text-color="#fff"
            active-text-color="#ffd04b">
            <el-submenu index="1">
              <template slot="title">
                <i class="el-icon-location"></i>
                <span>导航一</span>
              </template>
              <el-menu-item-group>
                <template slot="title">分组一</template>
                <el-menu-item index="1-1">选项1</el-menu-item>
                <el-menu-item index="1-2">选项2</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group title="分组2">
                <el-menu-item index="1-3">选项3</el-menu-item>
              </el-menu-item-group>
              <el-submenu index="1-4">
                <template slot="title">选项4</template>
                <el-menu-item index="1-4-1">选项1</el-menu-item>
              </el-submenu>
            </el-submenu>
            <el-menu-item index="2">
              <i class="el-icon-menu"></i>
              <span slot="title">导航二</span>
            </el-menu-item>
            <el-menu-item index="3" disabled>
              <i class="el-icon-document"></i>
              <span slot="title">导航三</span>
            </el-menu-item>
            <el-menu-item index="4">
              <i class="el-icon-setting"></i>
              <span slot="title">导航四</span>
            </el-menu-item>
          </el-menu>
  </el-col>
</el-row>
        </el-aside>
```

## 添加CSS样式

实现满屏的效果，需要在`.el-main`中设置：`min-height: calc(100vh - 120px);`

```css
html,
body {
  /* overflow-y: hidden; */
  padding: 0;
  margin: 0;
}
.el-header,
.el-footer {
  background-color: #b3c0d1;
  color: #333;
  text-align: center;
  height: 60px;
  line-height: 60px;
}

.el-aside {
  background-color: #d3dce6;
  color: #333;
  text-align: center;
  /* line-height: 200px; */
}
.el-main {
  background-color: #e9eef3;
  color: #333;
  text-align: center;
  line-height: 160px;
  min-height: calc(100vh - 120px);
}
```

效果：

![image-20201213195540608](https://i.loli.net/2020/12/13/c8iH19Fjy5xabpV.png)

## 启用router模式

> `router`参数：是否使用 vue-router 的模式，启用该模式会在激活导航时以 index 作为 path 进行路由跳转，布尔值。

在`el-menu`中设置`:router=true`，注意是动态属性，所以需要在前面加冒号：

```html
<el-menu
         :router=true
         :default-active="activeIndex2"
         class="el-menu-demo"
         mode="horizontal"
         @select="handleSelect"
         background-color="#545c64"
         text-color="#fff"
         active-text-color="#ffd04b"
  >
```

然后在标签中的`index`改成路由的路径：

```html
<el-menu-item index="/user">个人中心</el-menu-item>
```

这样就能点击渲染到相应的页面。

### 设置router模式后，设置默认激活菜单

`default-active`只支持字符串，所以需要用单引号把router路径包裹起来。

```html
 <el-menu
     :router=true
     :default-active="'/home'"
     class="el-menu-demo"
     mode="horizontal"
     @select="handleSelect"
     background-color="#545c64"
     text-color="#fff"
     active-text-color="#ffd04b"
 >
```

效果：

![image-20201215183351507](https://i.loli.net/2020/12/15/WGBrliuVUYjan1J.png)