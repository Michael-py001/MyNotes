## 配置Router

### 用vue-cli创建项目后，在router下的index.js文件中：

0. 如果使用模块化机制编程，导入Vue和VueRouter，要调用 Vue.use(VueRouter)

   ```js
   import Vue from 'vue'
   import VueRouter from 'vue-router'
   
   Vue.use(VueRouter) //全局使用，这样router可以在全局使用
   ```

1. 引入/定义(路由)组件

   ```js
   const Foo = { template: '<div>foo</div>' }
   //也可以从其他文件引入
   import Home from '../views/Home.vue';
   import my_app from '../views/my_app.vue'
   ```

2. 设置路由映射表，讲路由和组件的关系映射起来

   ```js
   const routes = [
    //①  
     {
       path: '/', //路径
       name: 'Home',  //命名路由
       component: Home //渲染的组件
     },
     //②
     {
       path: '/about',
       name: 'About',
       // route level code-splitting
       // this generates a separate chunk (about.[hash].js) for this route
       // which is lazy-loaded when the route is visited.
       component: () => import(/* webpackChunkName: "about" */ '../views/About.vue')
     },
     //③
     {
       path: "/my_app",
       name: "my_app",
       component: my_app
     }
       // 设置通配符，跳转到首页
     {
       path: '*',
       redirect:{
         path: '/home'
       }
     }
   ]
   ```

3. 创建router实例，传'routes'配置，将routes路由映射表放入router中进行渲染

   ```js
   const router = new VueRouter({
     mode: 'history', //history和hash模式
     base: process.env.BASE_URL,
     routes
   })
   //导出供main.js文件调用
   export default router
   ```

### 在根目录下的main.js主入口文件：

```js
import Vue from 'vue'
import App from './App.vue'  //引入App.vue模板
import router from './router' //引入路由配置

Vue.config.productionTip = false

// 4. 创建和挂载根实例。把router配置参数注入路由，从而让整个应用都有路由功能
new Vue({
  router,
  render: h => h(App)
}).$mount('#app')
```

### App.vue主模板文件

使用`router-link`、`router-view`：

`router-link`路由入口：使用 router-link 组件来导航，通过传入 `to` 属性指定链接，<router-link> 默认会被渲染成一个 `<a>` 标签

 		`to`可以设置成对象：

​			通过path或者name设置路径

```html
<router-link to={path:'/'}>Home</router-link> //使用path
<router-link to={name:'home'}>Home</router-link> //使用name
```

`router-view`路由出口：路由匹配到的组件将渲染在这里

```vue
<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link>|
      <router-link to="/my_app">my_home</router-link>
      <button v-on:click="goBack">回退</button>
    </div>
    <router-view />
  </div>
</template>
```

##  Router 构建选项

### 1. routes

传入的路由配置



### 2. mode 

- 类型: `string`

- 默认值: `"hash" (浏览器环境) | "abstract" (Node.js 环境)`

- 可选值: `"hash" | "history" | "abstract"`

  配置路由模式:

  - `hash`: 使用 URL hash 值来作路由。支持所有浏览器，包括不支持 HTML5 History Api 的浏览器。
  - `history`: 依赖 HTML5 History API 和服务器配置。查看 [HTML5 History 模式](https://router.vuejs.org/zh/guide/essentials/history-mode.html)。
  - `abstract`: 支持所有 JavaScript 运行环境，如 Node.js 服务器端。**如果发现没有浏览器的 API，路由会自动强制进入这个模式。**

### 3. base

- 类型: `string`

- 默认值: `"/"`

  应用的基路径。例如，如果整个单页应用服务在 `/app/` 下，然后 `base` 就应该设为 `"/app/"`。

- 根据环境：`process.env.BASE_URL`



###  4. linkActiveClass

- 类型: `string`

- 默认值: `"router-link-active"`

  全局配置 `<router-link>` 默认的激活的 class。

  

###  5. linkExactActiveClass

- 类型: `string`

- 默认值: `"router-link-exact-active"`

  全局配置 `<router-link>` 默认的精确激活的 class。

### active-class

- 类型: `string`

- 默认值: `"router-link-active"`

  设置链接激活时使用的 CSS 类名。默认值可以通过路由的构造选项 `linkActiveClass` 来全局配置。

### exact

- 类型: `boolean`

- 默认值: `false`

  “是否激活”默认类名的依据是**包含匹配**。 举个例子，如果当前的路径是 `/a` 开头的，那么 `<router-link to="/a">` 也会被设置 CSS 类名。

  按照这个规则，每个路由都会激活 `<router-link to="/">`！想要链接使用“精确匹配模式”，则使用 `exact` 属性：

  ```html
  <!-- 这个链接只会在地址为 / 的时候被激活 -->
  <router-link to="/" exact></router-link>
  ```

### 例子

在`App.vue`中设置css类名

```css
#nav a.active {
  font-size: 20px;
  font-weight: 600;
  color: rgb(59, 113, 196);
}
```

在router配置中，VueRouter构建项中添加：

```js
const router = new VueRouter({
  mode: 'history', //history和hash模式
  base: process.env.BASE_URL,
  routes,
  linkExactActiveClass: "active" //设置选中的css类名
})
```

## 动态路由匹配

我们经常需要把某种模式匹配到的所有路由，全都映射到同个组件。例如，我们有一个 `User` 组件，对于所有 ID 各不相同的用户，都要使用这个组件来渲染。那么，我们可以在 `vue-router` 的路由路径中使用“动态路径参数”(dynamic segment) 来达到这个效果：

```js
const User = {
  template: '<div>User</div>'
}

const router = new VueRouter({
  routes: [
    // 动态路径参数 以冒号开头
    { path: '/user/:id', component: User }
  ]
})
```

现在呢，像 `/user/foo` 和 `/user/bar` 都将映射到相同的路由。

一个“路径参数”使用冒号 `:` 标记。当匹配到一个路由时，参数值会被设置到 `this.$route.params`，可以在每个组件内使用。于是，我们可以更新 `User` 的模板，输出当前用户的 ID：

```js
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
```

你可以在一个路由中设置多段“路径参数”，对应的值都会设置到 `$route.params` 中。例如：

| 模式                          | 匹配路径            | $route.params                          |
| ----------------------------- | ------------------- | -------------------------------------- |
| /user/:username               | /user/evan          | `{ username: 'evan' }`                 |
| /user/:username/post/:post_id | /user/evan/post/123 | `{ username: 'evan', post_id: '123' }` |

除了 `$route.params` 外，`$route` 对象还提供了其它有用的信息，例如，`$route.query` (如果 URL 中有查询参数)、`$route.hash` 等等。你可以查看 [API 文档](https://router.vuejs.org/zh/api/#路由对象) 的详细说明。

或者使用 2.2 中引入的 `beforeRouteUpdate` [导航守卫](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)：

```js
const User = {
  template: '...',
  beforeRouteUpdate (to, from, next) {
    // react to route changes...
    // don't forget to call next()
  }
}
```

### [#](https://router.vuejs.org/zh/guide/essentials/dynamic-matching.html#捕获所有路由或-404-not-found-路由)捕获所有路由或 404 Not found 路由

常规参数只会匹配被 `/` 分隔的 URL 片段中的字符。如果想匹配**任意路径**，我们可以使用通配符 (`*`)：

```js
{
  // 会匹配所有路径
  path: '*'
}
{
  // 会匹配以 `/user-` 开头的任意路径
  path: '/user-*'
}
```

当使用*通配符*路由时，请确保路由的顺序是正确的，也就是说含有*通配符*的路由应该放在最后。路由 `{ path: '*' }` 通常用于客户端 404 错误。如果你使用了*History 模式*，请确保[正确配置你的服务器](https://router.vuejs.org/zh/guide/essentials/history-mode.html)。

当使用一个*通配符*时，`$route.params` 内会自动添加一个名为 `pathMatch` 参数。它包含了 URL 通过*通配符*被匹配的部分：

```js
// 给出一个路由 { path: '/user-*' }
this.$router.push('/user-admin')
this.$route.params.pathMatch // 'admin'
// 给出一个路由 { path: '*' }
this.$router.push('/non-existing')
this.$route.params.pathMatch // '/non-existing'
```

## 命名路由

### `router-link`对象方式匹配

​		`router-link`中的`to`，除了直接写`to="/"`以外，还可以用对象方式匹配，如根据`path`、`name`匹配：

```html
      <router-link :to="{ path: '/baner' }">baner</router-link>|
      <router-link :to="{ name: 'user' }">user</router-link>
```

有时候，通过一个名称来标识一个路由显得更方便一些，特别是在链接一个路由，或者是执行一些跳转的时候。你可以在创建 Router 实例的时候，在 `routes` 配置中给某个路由设置名称。

```js
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})
```

要链接到一个命名路由，可以给 `router-link` 的 `to` 属性传一个对象：

```html
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
```

这跟代码调用 `router.push()` 是一回事：

```js
router.push({ name: 'user', params: { userId: 123 }})
```

这两种方式都会把路由导航到 `/user/123` 路径。

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

### 例子：

> 效果：三个视图组件，第一个组件嵌套着一个子页面

`Vue.app`主模板：

```html
 <router-link to="/my_app">my_home</router-link>|
 <router-link :to="{ path: '/baner' }">baner</router-link>|
 <router-link :to="{ name: 'user' }">user</router-link>
```

路由配置：`index.js`:

```js
const router = new VueRouter({
  routes: [
    {
    path: "/my_app",
    name: "my_app",
    component: my_app,
    children: [
      {
        path: 'inside',
        component: inside
      }
    ]
  },
    {
      path: "/baner",
      name: "baner",
      component: baner
  },
  {
    path: "/user",
    name: "user",
    component: user
  }
  ]
})
```

`my_app`组件里嵌套着一个自己的`router-view`:

​	`router-link`的`to`指定是`my_app`下的`inside`子组件页面：

```html
<div id="nav">
      <router-link to="my_app/inside">my_home</router-link>|
</div>
<router-view />
```

效果：

![image-20201212182125047](https://i.loli.net/2020/12/12/mEHM82KvfPQlA1B.png)

## 重定向和别名

### [#](https://router.vuejs.org/zh/guide/essentials/redirect-and-alias.html#重定向)重定向

重定向也是通过 `routes` 配置来完成，下面例子是从 `/a` 重定向到 `/b`：

```js
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: '/b' }
  ]
})
```

重定向的目标也可以是一个命名的路由：

```js
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: { name: 'foo' }}
  ]
})
```

甚至是一个方法，动态返回重定向目标：

```js
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: to => {
      // 方法接收 目标路由 作为参数
      // return 重定向的 字符串路径/路径对象
    }}
  ]
})
```

注意[导航守卫](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)并没有应用在跳转路由上，而仅仅应用在其目标上。在下面这个例子中，为 `/a` 路由添加一个 `beforeEnter` 守卫并不会有任何效果。

其它高级用法，请参考[例子](https://github.com/vuejs/vue-router/blob/dev/examples/redirect/app.js)。

#### 设置通配符

```js
  // 设置通配符，跳转到首页
  {
    path: '*',
    redirect: {
      path: '/'
    }
  }
```

### [#](https://router.vuejs.org/zh/guide/essentials/redirect-and-alias.html#别名)别名

“重定向”的意思是，当用户访问 `/a`时，URL 将会被替换成 `/b`，然后匹配路由为 `/b`，那么“别名”又是什么呢？

**`/a` 的别名是 `/b`，意味着，当用户访问 `/b` 时，URL 会保持为 `/b`，但是路由匹配则为 `/a`，就像用户访问 `/a` 一样。**

上面对应的路由配置为：

```js
const router = new VueRouter({
  routes: [
    { path: '/a', component: A, alias: '/b' }
  ]
})
```

“别名”的功能让你可以自由地将 UI 结构映射到任意的 URL，而不是受限于配置的嵌套路由结构。

更多高级用法，请查看[例子](https://github.com/vuejs/vue-router/blob/dev/examples/route-alias/app.js)。