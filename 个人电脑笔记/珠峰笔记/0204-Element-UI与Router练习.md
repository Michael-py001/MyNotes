[TOC]



# Element-UI与Router练习

## 安装与引入Element-UI

### 安装

```
npm i element-ui -S
```

### 完整引入

在 main.js 中写入以下内容：

```javascript
import Vue from 'vue'
import App from './App.vue'
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
// 4. 引入路由配置，并在Vue里传入使用
import router from './router'
Vue.use(ElementUI);

new Vue({
  router,
  render: h => h(App)
}).$mount('#app')

```

### 按需引入

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

完整组件列表和引入方式请看官网：[https://element.eleme.cn/#/zh-CN/component/quickstart][https://element.eleme.cn/#/zh-CN/component/quickstart]

## Container 布局容器

[https://element.eleme.cn/#/zh-CN/component/container][https://element.eleme.cn/#/zh-CN/component/container]

用于布局的容器组件，方便快速搭建页面的基本结构：

`<el-container>`：外层容器。当子元素中包含 `<el-header>` 或 `<el-footer>` 时，全部子元素会垂直上下排列，否则会水平左右排列。

`<el-header>`：顶栏容器。

`<el-aside>`：侧边栏容器。

`<el-main>`：主要区域容器。

`<el-footer>`：底栏容器。



使用一个常见的布局：

```html
<el-container>
    <el-aside width="200px">Aside</el-aside>
    <el-container>
        <el-header>Header</el-header>
        <el-main>Main</el-main>
        <el-footer>Footer</el-footer>
    </el-container>
</el-container>
```

要实现全屏显示，需要给main添加CSS的calc属性: **100vh(全屏的高度) - header和footer的高度(60px+60px=120px)**

```css
.el-header,
.el-footer {
  background-color: #b3c0d1;
  color: #333;
  text-align: center;
  line-height: 60px;
  height: 60px;
}

.el-aside {
  background-color: #d3dce6;
  color: #333;
  text-align: center;
  min-height: 100vh;
}

.el-main {
  background-color: #e9eef3;
  color: #333;
  text-align: center;
  min-height: calc(100vh - 120px);
}
```

![image-20210206174534530](https://i.loli.net/2021/02/18/CiegIFYkz7xwU93.png)

## NavMenu 导航菜单

> https://element.eleme.cn/#/zh-CN/component/menu

### 顶栏 

![image-20210218180441766](https://i.loli.net/2021/02/18/pJZFCEr4SaAN2gW.png)

```vue
  <el-container>
      <el-aside width="200px">Aside</el-aside>
      <el-container>
        <el-header>
          <el-menu
            :router="true"
            :default-active="this.$route.path"
            class="el-menu-demo"
            mode="horizontal"
            @select="handleSelect"
            background-color="#545c64"
            text-color="#fff"
            active-text-color="#ffd04b"
          >
            <el-menu-item index="/home">首页</el-menu-item>
            <el-menu-item index="/profile">个人中心</el-menu-item>
            <el-menu-item index="/user">用户管理</el-menu-item>
          </el-menu>
          <div class="line"></div>
        </el-header>
        <el-main>
          <router-view></router-view>
        </el-main>
        <el-footer>Footer</el-footer>
      </el-container>
    </el-container>
```

- 开启router模式，需要在属性里加`:router="true"`，注意是前面加冒号，动态属性。接着把`el-menu-item`中的`index`属性改为路由的`path`路径。

- 开启路由后，设置页面刷新激活菜单保持不变：把`:default-active`的值改成`:default-active="this.$route.path"`，也就是通过`$route.path`动态获取当前的路径。

## 嵌套路由

实际生活中的应用界面，通常由多层嵌套的组件组合而成。同样地，URL 中各段动态路径也按某种结构对应嵌套的各层组件，例如：

```text
/user/foo/profile                     /user/foo/posts
+------------------+                  +-----------------+
| User             |                  | User            |
| +--------------+ |                  | +-------------+ |
| | Profile      | |  +------------>  | | Posts       | |
| |              | |                  | |             | |
| +--------------+ |                  | +-------------+ |
+------------------+                  +-----------------+
```

借助 `vue-router`，使用嵌套路由配置，就可以很简单地表达这种关系。

接着上节创建的 app：

```html
<div id="app">
  <router-view></router-view>
</div>
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}

const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User }
  ]
})
```

这里的 `<router-view>` 是最顶层的出口，渲染最高级路由匹配到的组件。同样地，一个被渲染组件同样可以包含自己的嵌套 `<router-view>`。例如，在 `User` 组件的模板添加一个 `<router-view>`：

```js
const User = {
  template: `
    <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view>
    </div>
  `
}
```

要在嵌套的出口中渲染组件，需要在 `VueRouter` 的参数中使用 `children` 配置：

```js
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          component: UserProfile
        },
        {
          // 当 /user/:id/posts 匹配成功
          // UserPosts 会被渲染在 User 的 <router-view> 中
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
```

**要注意，以 `/` 开头的嵌套路径会被当作根路径。 这让你充分的使用嵌套组件而无须设置嵌套的路径。**

你会发现，`children` 配置就是像 `routes` 配置一样的路由配置数组，所以呢，你可以嵌套多层路由。

此时，基于上面的配置，当你访问 `/user/foo` 时，`User` 的出口是不会渲染任何东西，这是因为没有匹配到合适的子路由。如果你想要渲染点什么，可以提供一个 空的 子路由：

```js
const router = new VueRouter({
  routes: [
    {
      path: '/user/:id', component: User,
      children: [
        // 当 /user/:id 匹配成功，
        // UserHome 会被渲染在 User 的 <router-view> 中
        { path: '', component: UserHome },

        // ...其他子路由
      ]
    }
  ]
})
```

提供以上案例的可运行代码请[移步这里](https://jsfiddle.net/yyx990803/L7hscd8h/)。

### 嵌套路由按需引入

```js
 {
    path: '/user',
    name: 'user',
    component: () => import(/* webpackChunkName: "user" */ '@/views/User.vue'),
    children: [
      {
        path: 'add',
        compoment: () => import('@/views/UserAdd') //按需引入
      },
      {
        path: 'list',
        compoment: () => import('@/views/UserList')
      }
    ]
  },
```

## 嵌套路由使用

### 二级路由的侧边栏

![image-20210219021342075](https://i.loli.net/2021/02/19/VFOdWSiXHCaDm2w.png)

只需要把刚刚的导航代码中的mode模式改为`vertical`，即可竖着显示。

```html
  <el-container>
      <el-aside>
        <el-row>
          <el-col :span="24">
            <!-- 二级路由的侧边栏 -->
            <el-menu
              :router="true"
              mode="vertical"
              :default-active="
                this.$route.path === '/user' ? '/user/add' : this.$route.path
              "
              background-color="#545c64"
              text-color="#fff"
              active-text-color="#ffd04b"
            >
              <el-menu-item index="/user/add">
                <i class="el-icon-location"></i>
                <span slot="title">用户添加</span>
              </el-menu-item>

              <el-menu-item index="/user/list">
                <i class="el-icon-menu"></i>
                <span slot="title">用户列表</span>
              </el-menu-item>
            </el-menu>
          </el-col>
        </el-row>
      </el-aside>
      <el-main>
        <!-- 二级路由的视图显示 -->
        <router-view></router-view>
      </el-main>
    </el-container>
```

### 激活菜单的选中问题

1. 在图中的二级路由中的侧边栏菜单，用户选中时，一级路由的选中状态会消失，这时候需要手动进行修改设置，把App.vue中的`el-menu`中的`defalut-active`属性进行判断：

   而且，需要默认显示用户添加页面，即`user/add` 。

   ```html
    <el-menu
               :default-active="
                 this.$route.path.includes('user') ? '/user/add' : this.$route.path
               "
               :router="true"
             >
   ```

2. 点击用户管理这个菜单时，下面的侧边栏菜单不会默认显示，需要手动进行设置：

   ```html
    <el-menu
               :router="true"
               mode="vertical"
               :default-active="
                 this.$route.path === '/user' ? '/user/add' : this.$route.path
               "
             >
   ```

   

## 编程式的导航-路由的跳转

除了使用 `<router-link>` 创建 a 标签来定义导航链接，我们还可以借助 router 的实例方法，通过编写代码来实现。

### [#](https://router.vuejs.org/zh/guide/essentials/navigation.html#router-push-location-oncomplete-onabort)`router.push(location, onComplete?, onAbort?)`

**注意：在 Vue 实例内部，你可以通过 `$router` 访问路由实例。因此你可以调用 `this.$router.push`。**

想要导航到不同的 URL，则使用 `router.push` 方法。这个方法会向 history 栈添加一个新的记录，所以，当用户点击浏览器后退按钮时，则回到之前的 URL。

当你点击 `<router-link>` 时，这个方法会在内部调用，所以说，点击 `<router-link :to="...">` 等同于调用 `router.push(...)`。

| 声明式                    | 编程式             |
| ------------------------- | ------------------ |
| `<router-link :to="...">` | `router.push(...)` |

该方法的参数可以是一个字符串路径，或者一个描述地址的对象。例如：

```js
// 字符串
router.push('home')

// 对象
router.push({ path: 'home' })

// 命名的路由
router.push({ name: 'user', params: { userId: '123' }})

// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }})
```

**注意：如果提供了 `path`，`params` 会被忽略，上述例子中的 `query` 并不属于这种情况。取而代之的是下面例子的做法，你需要提供路由的 `name` 或手写完整的带有参数的 `path`：**

```js
const userId = '123'
router.push({ name: 'user', params: { userId }}) // -> /user/123
router.push({ path: `/user/${userId}` }) // -> /user/123
// 这里的 params 不生效
router.push({ path: '/user', params: { userId }}) // -> /user
```

同样的规则也适用于 `router-link` 组件的 `to` 属性。

在 2.2.0+，可选的在 `router.push` 或 `router.replace` 中提供 `onComplete` 和 `onAbort` 回调作为第二个和第三个参数。这些回调将会在导航成功完成 (在所有的异步钩子被解析之后) 或终止 (导航到相同的路由、或在当前导航完成之前导航到另一个不同的路由) 的时候进行相应的调用。在 3.1.0+，可以省略第二个和第三个参数，此时如果支持 Promise，`router.push` 或 `router.replace` 将返回一个 Promise。

**注意**： 如果目的地和当前路由相同，只有参数发生了改变 (比如从一个用户资料到另一个 `/users/1` -> `/users/2`)，你需要使用 [`beforeRouteUpdate`](https://router.vuejs.org/zh/guide/essentials/dynamic-matching.html#响应路由参数的变化) 来响应这个变化 (比如抓取用户信息)。

### [#](https://router.vuejs.org/zh/guide/essentials/navigation.html#router-replace-location-oncomplete-onabort)`router.replace(location, onComplete?, onAbort?)`

跟 `router.push` 很像，唯一的不同就是，它不会向 history 添加新记录，而是跟它的方法名一样 —— 替换掉当前的 history 记录。

| 声明式                            | 编程式                |
| --------------------------------- | --------------------- |
| `<router-link :to="..." replace>` | `router.replace(...)` |

### [#](https://router.vuejs.org/zh/guide/essentials/navigation.html#router-go-n)`router.go(n)`

这个方法的参数是一个整数，意思是在 history 记录中向前或者后退多少步，类似 `window.history.go(n)`。

例子

```js
// 在浏览器记录中前进一步，等同于 history.forward()
router.go(1)

// 后退一步记录，等同于 history.back()
router.go(-1)

// 前进 3 步记录
router.go(3)

// 如果 history 记录不够用，那就默默地失败呗
router.go(-100)
router.go(100)
```

## [#](https://router.vuejs.org/zh/guide/essentials/navigation.html#操作-history)操作 History

你也许注意到 `router.push`、 `router.replace` 和 `router.go` 跟 [`window.history.pushState`、 `window.history.replaceState` 和 `window.history.go`](https://developer.mozilla.org/en-US/docs/Web/API/History)好像， 实际上它们确实是效仿 `window.history` API 的。

因此，如果你已经熟悉 [Browser History APIs](https://developer.mozilla.org/en-US/docs/Web/API/History_API)，那么在 Vue Router 中操作 history 就是超级简单的。

还有值得提及的，Vue Router 的导航方法 (`push`、 `replace`、 `go`) 在各类路由模式 (`history`、 `hash` 和 `abstract`) 下表现一致。

## History API

DOM [`window`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window) 对象通过 [`history`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/history) 对象提供了对浏览器的会话历史的访问(不要与 [WebExtensions history](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/history)搞混了)。它暴露了很多有用的方法和属性，允许你在用户浏览历史中向前和向后跳转，同时——从HTML5开始——提供了对history栈中内容的操作。

### [意义及使用](https://developer.mozilla.org/zh-CN/docs/Web/API/History_API#意义及使用)

使用 [`back()`](https://developer.mozilla.org/zh-CN/docs/Web/API/History/back),  [`forward()`](https://developer.mozilla.org/zh-CN/docs/Web/API/History/forward)和  [`go()`](https://developer.mozilla.org/zh-CN/docs/Web/API/History/go) 方法来完成在用户历史记录中向后和向前的跳转。

### [向前和向后跳转](https://developer.mozilla.org/zh-CN/docs/Web/API/History_API#向前和向后跳转)

在history中向后跳转：

```
window.history.back();
```

这和用户点击浏览器回退按钮的效果相同。

类似地，你可以向前跳转（如同用户点击了前进按钮）：

```
window.history.forward();
```

### [跳转到 history 中指定的一个点](https://developer.mozilla.org/zh-CN/docs/Web/API/History_API#跳转到_history_中指定的一个点)

你可以用 `go()` 方法载入到会话历史中的某一特定页面， 通过与当前页面相对位置来标志 (当前页面的相对位置标志为0).

向后移动一个页面 (等同于调用 `back()`):

```
window.history.go(-1);
```

向前移动一个页面, 等同于调用了 `forward()`:

```
window.history.go(1);
```

类似地，你可以传递参数值2并向前移动2个页面，等等。

您可以通过查看长度属性的值来确定的历史堆栈中页面的数量:

```
 let numberOfEntries = window.history.length;
```

## 实现用户添加和显示功能

### 用户添加-form表单组件

> https://element.eleme.cn/2.15/#/zh-CN/component/form

```vue
//userAdd.vue
<template>
  <div>
    <el-form :model="ruleForm" :rules="rules" ref="form">
      <el-form-item label="用户名" prop="username">
        <el-input v-model="ruleForm.username"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="addUser">立即添加</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>
<script>
export default {
  data() {
    return {
      ruleForm: {
        username: "",
      },
      rules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" },
          {
            min: 3,
            max: 15,
            message: "长度在 3 到 15 个字符",
            trigger: "blur",
          },
        ],
      },
    };
  },

  methods: {
    addUser(form) {
      // this.$refs[form].validate 也可以
      this.$refs.form.validate((valid) => {
        if (valid) {
          let id = Math.random();
          // 存到localstrorage中，先判断是否存在lists
          let lists = JSON.parse(localStorage.getItem("lists")) || [];
          // 把data中的ruleForm数据进去
          lists.push({ id, username: this.ruleForm.username });
          //存到localstrorage
          localStorage.setItem("lists", JSON.stringify(lists));
          alert("添加用户成功!");
          // 路由跳转到显示列表
          this.$router.push("/user/list");
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
  },
};
</script>
```

![image-20210223230343375](https://i.loli.net/2021/02/23/UfJEPwzYeKixbrh.png)

### localStorage语法

```js
window.localStorage
```

保存数据语法：

```js
localStorage.setItem("key", "value");
```

读取数据语法：

```js
var lastname = localStorage.getItem("key");
```

删除数据语法：

```js
localStorage.removeItem("key");
```

### 用户显示-table组件

> https://element.eleme.cn/2.15/#/zh-CN/component/table

通过 `Scoped slot` 可以获取到 row, column, $index 和 store（table 内部的状态管理）的数据，用法参考下面的 demo。

```vue
//**userList.vue**
<template>
  <div>
    <el-table :data="userData" style="width: 100%">
      <el-table-column prop="id" label="ID" width="180"> </el-table-column>
      <el-table-column
        prop="username"
        label="用户名"
        width="440"
      ></el-table-column>
      <el-table-column fixed="right" label="操作" width="180">
        <template slot-scope="scope">
          <!-- 通过 Scoped slot 可以获取到 row, column, $index 和 store（table 内部的状态管理）的数据 -->
          <el-button
            type="text"
            size="small"
            @click.native.prevent="deleteRow(scope.$index, userData)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>
<script>
export default {
  data() {
    let userData = JSON.parse(localStorage.getItem("lists"));
    return {
      userData,
    };
  },
  methods: {
    deleteRow(index, data) {
      let userData = JSON.parse(localStorage.getItem("lists"));
      // 把本地的数据删除一个
      userData.splice(index, 1);
      // 重新写入数据到本地
      localStorage.setItem("lists", JSON.stringify(userData));
      // 第二个参数传进来的参数就是userData的数据
      data.splice(index, 1);
    },
  },
};
</script>
```

![image-20210223230318139](https://i.loli.net/2021/02/23/egHhfJnMPO4pUza.png)

## 路由对象

一个**路由对象 (route object)** 表示当前激活的路由的状态信息，包含了当前 URL 解析得到的信息，还有 URL 匹配到的**路由记录 (route records)**。

路由对象是不可变 (immutable) 的，每次成功的导航后都会产生一个新的对象。

路由对象出现在多个地方:

- 在组件内，即 `this.$route`

- 在 `$route` 观察者回调内

- `router.match(location)` 的返回值

- 导航守卫的参数：

  ```js
  router.beforeEach((to, from, next) => {
    // `to` 和 `from` 都是路由对象
  })
  ```

- `scrollBehavior` 方法的参数:

  ```js
  const router = new VueRouter({
    scrollBehavior(to, from, savedPosition) {
      // `to` 和 `from` 都是路由对象
    }
  })
  ```

### [#](https://router.vuejs.org/zh/api/#路由对象属性)路由对象属性

- **$route.path**

  - 类型: `string`

    字符串，对应当前路由的路径，总是解析为绝对路径，如 `"/foo/bar"`。

- **$route.params**

  - 类型: `Object`

    一个 key/value 对象，包含了动态片段和全匹配片段，如果没有路由参数，就是一个空对象。

- **$route.query**

  - 类型: `Object`

    一个 key/value 对象，表示 URL 查询参数。例如，对于路径 `/foo?user=1`，则有 `$route.query.user == 1`，如果没有查询参数，则是个空对象。

- **$route.hash**

  - 类型: `string`

    当前路由的 hash 值 (带 `#`) ，如果没有 hash 值，则为空字符串。

- **$route.fullPath**

  - 类型: `string`

    完成解析后的 URL，包含查询参数和 hash 的完整路径。

- **$route.matched**

  - 类型: `Array<RouteRecord>`

  一个数组，包含当前路由的所有嵌套路径片段的**路由记录** 。路由记录就是 `routes` 配置数组中的对象副本 (还有在 `children` 数组)。

  ```js
  const router = new VueRouter({
    routes: [
      // 下面的对象就是路由记录
      {
        path: '/foo',
        component: Foo,
        children: [
          // 这也是个路由记录
          { path: 'bar', component: Bar }
        ]
      }
    ]
  })
  ```

  当 URL 为 `/foo/bar`，`$route.matched` 将会是一个包含从上到下的所有对象 (副本)。

- **$route.name**

  当前路由的名称，如果有的话。(查看[命名路由](https://router.vuejs.org/zh/guide/essentials/named-routes.html))

- **$route.redirectedFrom**

  如果存在重定向，即为重定向来源的路由的名字。(参阅[重定向和别名](https://router.vuejs.org/zh/guide/essentials/redirect-and-alias.html))

## Vue中使用less踩坑

### 安装less

```
npm i less less-loader -S
```

开始使用less时(vue-cli 4.5)，编译出现报错，经过一番捣鼓后才知道，原来是less-loader的版本太贵(8.0+)，解决办法很简单，降级为7.x版本就行了。

```
npm uninstall less-loader -S

npm i less-loader@7.x -S
```

## Table 表格

> 用于展示多条结构类似的数据，可对数据进行排序、筛选、对比或其他自定义操作。

```html
 <el-table :data="userData" style="width: 100%">
      <el-table-column prop="id" label="ID" width="180"> </el-table-column>
      <el-table-column
        prop="username"
        label="用户名"
        width="440"
      ></el-table-column>
      <el-table-column fixed="right" label="操作" width="160">
        <template slot-scope="scope">
          <el-button size="mini" @click="handleEdit(scope.$index, scope.row)"
            >编辑</el-button
          >
          <!-- 通过 Scoped slot 可以获取到 row, column, $index 和 store（table 内部的状态管理）的数据 -->
          <el-button
            type="danger"
            size="mini"
            @click.native.prevent="
              deleteRow(
                scope.$index,
                scope.row,
                scope.column,
                scope.store,
                userData
              )
            "
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
```

### slot-scope-作用域插槽

> 携带数据的插槽，叫作用域插槽。通过 Scoped slot 可以获取到 row, column, $index 和 store（table 内部的状态管理）的数据。也就是一整行的所有数据。

```html
<el-table :data="userData" style="width: 100%">
      <el-table-column prop="id" label="ID" width="180"> </el-table-column>
      <el-table-column
        prop="username"
        label="用户名"
        width="440"
      ></el-table-column>
      <el-table-column fixed="right" label="操作" width="160">
          <!-- 作用域插槽 -->
        <template slot-scope="scope">
          <el-button size="mini" @click="handleEdit(scope.$index, scope.row)"
            >编辑</el-button
          >
          <!-- 通过 Scoped slot 可以获取到 row, column, $index 和 store（table 内部的状态管理）的数据 -->
          <el-button
            type="danger"
            size="mini"
            @click.native.prevent="
              deleteRow(
                scope.$index,
                scope.row,
                scope.column,
                scope.store,
                userData
              )
            "
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
```



## MessageBox 弹框

> https://element.eleme.cn/2.15/#/zh-CN/component/message-box

```js
 this.$confirm("确认删除吗?", "提示", {})
        .then(() => {
          let userData = JSON.parse(localStorage.getItem("lists"));
          // 把本地的数据删除一个
          userData.splice(index, 1);
          // 重新写入数据到本地
          localStorage.setItem("lists", JSON.stringify(userData));
          // 第二个参数传进来的参数就是userData的数据
          data.splice(index, 1);
          this.$message({
            message: "删除成功",
            type: "success",
            center: true,
          });
        })
        .catch(() => {
          /* 用户取消删除 */
          this.$message({
            type: "info",
            message: "已取消删除",
            center: true,
          });
        });
```



### element-ui 消息提示框样式修改

可以覆盖原有的样式，也可以选择`customClass`自定义类名：

```js
this.$message({
            message: "添加用户成功!",
            type: "success",
            center: true,
            duration: 3000,
            showClose: true,
            customClass: "tips",
          });
```



```css
<style >
/*alert 成功弹出框样式*/

/* 自定义类名 */
.tips {
  min-width: 300px;
}
/* 覆盖原有类名 */
.el-message {
  min-width: 300px;
}
.el-message--success {
  top: 15px !important;
  max-width: 100px !important;
}

.el-message .el-icon-success {
  font-size: 18px;
}

.el-message--success .el-message__content {
  font-size: 18px;
  font-weight: 700;
}
</style>
```

## router-link传参

> 用法与`this.$router.push()`一致

### 第一种：路由传参

```html
<router-link type="text" :to="`/user/edit/${scope.$index}`">编辑</router-link>
```

注意这里`:to`里传的参数名称要与路由配置的`path`路径一致，都是`index`

路由配置：

```js
{
    path: 'edit/:index',
    component: () => import('@/views/UserEdit.vue')
}
```

缺点：只能传一个参数

### 第二种:问号传参

```html
<router-link type="text" :to="`/user/edit?index=${index}&id=${id}&username=${username}`">编辑</router-link>
```

路由配置：

```js
 {
        path: 'edit',
        component: () => import('@/views/UserEdit.vue')
      }
```

