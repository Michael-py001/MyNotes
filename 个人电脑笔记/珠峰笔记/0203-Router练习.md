# Router练习

## 创建项目

```
vue create learn_router
```

选择router功能后，会有一个`view`的文件夹。

![image-20210203002620223](https://i.loli.net/2021/02/03/OXMds1SuK34FWLE.png)

- `views`文件夹：一般放视图组件；
- `components`文件夹：一般放公共组件（如公用的头部，尾部，侧边栏）
- `router`文件夹：设置路由
- `main.js`: 默认的入口文件
- `App.vue`：是所有组件的入口文件

## Router配置

三大步骤：

1. 引入组件
2. 设置路由映射表，把组件和路由的关系映射起来
3. 将routes放入router中进行渲染
4. 在主入口`main.js`中引入router配置并在vue中传入使用

配置路由：

```js
import Vue from 'vue'
import VueRouter from 'vue-router'
// 配置路由三大步骤：

// 1、引入组件
import Home from '../views/Home.vue'

Vue.use(VueRouter) //全局使用，所有组件都能使用router

// 2. 设置路由映射表，把组件和路由的关系映射起来
const routes = [
  {
    path: '/',
    name: 'Home', //命名路由，可以通过路由的名字进行跳转
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    component: () => import(/* webpackChunkName: "about" */ '../views/About.vue')
  }
]
// 3.将routes放入router中进行渲染
const router = new VueRouter({
 mode: 'history', //hash
  base: process.env.BASE_URL, //应用的基路径
  routes,
  // linkActiveClass:"aaa"  默认激活的class类名
})

export default router
```

在`main.js`中使用Router：

```js
import Vue from 'vue'
import App from './App.vue'
// 4. 引入路由配置，并在Vue里传入使用
import router from './router'

Vue.config.productionTip = false

new Vue({
  router,
  render: h => h(App)
}).$mount('#app')
```



## 命名路由

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

完整的例子请[移步这里](https://github.com/vuejs/vue-router/blob/dev/examples/named-routes/app.js)。

## Router构建选项

### routes

- 类型: `Array<RouteConfig>`

  `RouteConfig` 的类型定义：

  ```ts
  interface RouteConfig = {
    path: string,
    component?: Component,
    name?: string, // 命名路由
    components?: { [name: string]: Component }, // 命名视图组件
    redirect?: string | Location | Function,
    props?: boolean | Object | Function,
    alias?: string | Array<string>,
    children?: Array<RouteConfig>, // 嵌套路由
    beforeEnter?: (to: Route, from: Route, next: Function) => void,
    meta?: any,
  
    // 2.6.0+
    caseSensitive?: boolean, // 匹配规则是否大小写敏感？(默认值：false)
    pathToRegexpOptions?: Object // 编译正则的选项
  }
  ```

### mode

- 类型: `string`

- 默认值: `"hash" (浏览器环境) | "abstract" (Node.js 环境)`

- 可选值: `"hash" | "history" | "abstract"`

  配置路由模式:

  - `hash`: 使用 URL hash 值来作路由。支持所有浏览器，包括不支持 HTML5 History Api 的浏览器。
  - `history`: 依赖 HTML5 History API 和服务器配置。查看 [HTML5 History 模式](https://router.vuejs.org/zh/guide/essentials/history-mode.html)。
  - `abstract`: 支持所有 JavaScript 运行环境，如 Node.js 服务器端。**如果发现没有浏览器的 API，路由会自动强制进入这个模式。**

#### HTML5 History 模式

`vue-router` 默认 hash 模式 —— 使用 URL 的 hash 来模拟一个完整的 URL，于是当 URL 改变时，页面不会重新加载。

如果不想要很丑的 hash，我们可以用路由的 **history 模式**，这种模式充分利用 `history.pushState` API 来完成 URL 跳转而无须重新加载页面。

```js
const router = new VueRouter({
  mode: 'history',
  routes: [...]
})
```

当你使用 history 模式时，URL 就像正常的 url，例如 `http://yoursite.com/user/id`，也好看！

不过这种模式要玩好，还需要后台配置支持。因为我们的应用是个单页客户端应用，如果后台没有正确的配置，当用户在浏览器直接访问 `http://oursite.com/user/id` 就会返回 404，这就不好看了。

所以呢，你要在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 `index.html` 页面，这个页面就是你 app 依赖的页面。

### base

- 类型: `string`

- 默认值: `"/"`

  应用的基路径。例如，如果整个单页应用服务在 `/app/` 下，然后 `base` 就应该设为 `"/app/"`。

### [#](https://router.vuejs.org/zh/api/#linkactiveclass)linkActiveClass

- 类型: `string`

- 默认值: `"router-link-active"`

  全局配置 `<router-link>` **默认**的激活的 class。参考 [router-link](https://router.vuejs.org/zh/api/#router-link)。

  > linkActiveClass 是指匹配到前面有相同的就会激活，比如'/user' ，此时'/'和'/user'会同时激活，所以需要精确匹配时，最好用linkExactActiveClass.

### [#](https://router.vuejs.org/zh/api/#linkexactactiveclass)linkExactActiveClass

- 类型: `string`

- 默认值: `"router-link-exact-active"`

  全局配置 `<router-link>` **默认**的精确激活的 class。可同时翻阅 [router-link](https://router.vuejs.org/zh/api/#router-link)。

## `<router-link>` Props

### to

- 类型: `string | Location`

- required

  表示目标路由的链接。当被点击后，内部会立刻把 `to` 的值传到 `router.push()`，所以这个值可以是一个字符串或者是描述目标位置的对象。

  ```html
  <!-- 字符串 -->
  <router-link to="home">Home</router-link>
  <!-- 渲染结果 -->
  <a href="home">Home</a>
  
  <!-- 使用 v-bind 的 JS 表达式 -->
  <router-link v-bind:to="'home'">Home</router-link>
  
  <!-- 不写 v-bind 也可以，就像绑定别的属性一样 -->
  <router-link :to="'home'">Home</router-link>
  
  <!-- 同上 -->
  <router-link :to="{ path: 'home' }">Home</router-link>
  
  <!-- 命名的路由 -->
  <router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
  
  <!-- 带查询参数，下面的结果为 /register?plan=private -->
  <router-link :to="{ path: 'register', query: { plan: 'private' }}"
    >Register</router-link
  >
  ```

### [#](https://router.vuejs.org/zh/api/#replace)replace

- 类型: `boolean`

- 默认值: `false`

  设置 `replace` 属性的话，当点击时，会调用 `router.replace()` 而不是 `router.push()`，于是导航后不会留下 history 记录。

  ```html
  <router-link :to="{ path: '/abc'}" replace></router-link>
  ```

### [#](https://router.vuejs.org/zh/api/#append)append

- 类型: `boolean`

- 默认值: `false`

  设置 `append` 属性后，则在当前 (相对) 路径前添加基路径。例如，我们从 `/a` 导航到一个相对路径 `b`，如果没有配置 `append`，则路径为 `/b`，如果配了，则为 `/a/b`

  ```html
  <router-link :to="{ path: 'relative/path'}" append></router-link>
  ```

### [#](https://router.vuejs.org/zh/api/#tag)tag

- 类型: `string`

- 默认值: `"a"`

  有时候想要 `<router-link>` 渲染成某种标签，例如 `<li>`。 于是我们使用 `tag` prop 类指定何种标签，同样它还是会监听点击，触发导航。

  ```html
  <router-link to="/foo" tag="li">foo</router-link>
  <!-- 渲染结果 -->
  <li>foo</li>
  ```

### [#](https://router.vuejs.org/zh/api/#active-class)active-class

- 类型: `string`

- 默认值: `"router-link-active"`

  设置链接激活时使用的 CSS 类名。默认值可以通过路由的构造选项 `linkActiveClass` 来全局配置。

### [#](https://router.vuejs.org/zh/api/#exact)exact

- 类型: `boolean`

- 默认值: `false`

  “是否激活”默认类名的依据是**包含匹配**。 举个例子，如果当前的路径是 `/a` 开头的，那么 `<router-link to="/a">` 也会被设置 CSS 类名。

  按照这个规则，每个路由都会激活 `<router-link to="/">`！想要链接使用“精确匹配模式”，则使用 `exact` 属性：

  ```html
  <!-- 这个链接只会在地址为 / 的时候被激活 -->
  <router-link to="/" exact></router-link>
  ```

  查看更多关于激活链接类名的例子[可运行](https://jsfiddle.net/8xrk1n9f/)

### [#](https://router.vuejs.org/zh/api/#event)event

- 类型: `string | Array<string>`

- 默认值: `'click'`

  声明可以用来触发导航的事件。可以是一个字符串或是一个包含字符串的数组。

### [#](https://router.vuejs.org/zh/api/#exact-active-class)exact-active-class

- 类型: `string`

- 默认值: `"router-link-exact-active"`

  配置当链接被精确匹配的时候应该激活的 class。注意默认值也是可以通过路由构造函数选项 `linkExactActiveClass` 进行全局配置的。

例子：

```html
<router-link to="/user" exact-active-class="active">用户</router-link>
```

```css
.active {
  color: greenyellow;
}
```

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

或者以对象的形式：

```js
const routes = [
  {
    path: '*',
    redirect: {path:'/'} 
  }
]
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

## [#](https://router.vuejs.org/zh/guide/essentials/named-views.html#命名视图)命名视图

有时候想同时 (同级) 展示多个视图，而不是嵌套展示，例如创建一个布局，有 `sidebar` (侧导航) 和 `main` (主内容) 两个视图，这个时候命名视图就派上用场了。你可以在界面中拥有多个单独命名的视图，而不是只有一个单独的出口。如果 `router-view` 没有设置名字，那么默认为 `default`。

```html
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <h1>Named Views</h1>
  <ul>
    <li>
      <router-link to="/">/</router-link>
    </li>
    <li>
      <router-link to="/other">/other</router-link>
    </li>
  </ul>
  <router-view class="view one"></router-view>
  <router-view class="view two" name="a"></router-view>
  <router-view class="view three" name="b"></router-view>
</div>

```

一个视图使用一个组件渲染，因此对于同个路由，多个视图就需要多个组件。确保正确使用 `components` 配置 (带上 s)：

```js
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }
const Baz = { template: '<div>baz</div>' }

const router = new VueRouter({
  mode: 'history',
  routes: [
    { path: '/',
      // a single route can define multiple named components
      // which will be rendered into <router-view>s with corresponding names.
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    },
    {
      path: '/other',
      components: {
        default: Baz,
        a: Bar,
        b: Foo
      }
    }
  ]
})

new Vue({
	router,
  el: '#app'
})

```

效果：

![嵌套视图](https://i.loli.net/2021/02/04/91zioGxFms26HvU.gif)

以上案例相关的可运行代码请[移步这里](https://jsfiddle.net/posva/6du90epg/)。

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

你可以看看这个[在线例子](https://jsfiddle.net/yyx990803/4xfa2f19/)。

html:

```html
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <p>
    <router-link to="/user/foo">/user/foo</router-link>
    <router-link to="/user/bar">/user/bar</router-link>
  </p>
  <router-view></router-view>
</div>
```

JS:

```js
const User = {
  template: `<div>User {{ $route.params.id }}</div>`
}

const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User }
  ]
})

const app = new Vue({ router }).$mount('#app')
```

效果：

![动态参数](https://i.loli.net/2021/02/04/D8pyBwqiO2SmePs.gif)

你可以在一个路由中设置多段“路径参数”，对应的值都会设置到 `$route.params` 中。例如：

| 模式                          | 匹配路径            | $route.params                          |
| ----------------------------- | ------------------- | -------------------------------------- |
| /user/:username               | /user/evan          | `{ username: 'evan' }`                 |
| /user/:username/post/:post_id | /user/evan/post/123 | `{ username: 'evan', post_id: '123' }` |

除了 `$route.params` 外，`$route` 对象还提供了其它有用的信息，例如，`$route.query` (如果 URL 中有查询参数)、`$route.hash` 等等。你可以查看 [API 文档](https://router.vuejs.org/zh/api/#路由对象) 的详细说明。

## [#](https://router.vuejs.org/zh/guide/essentials/nested-routes.html#嵌套路由)嵌套路由

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

**要在嵌套的出口中渲染组件**，需要在 `VueRouter` 的参数中使用 `children` 配置：

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

此时，基于上面的配置，当你访问 `/user/foo` 时，`User` 的出口是不会渲染任何东西，这是因为没有匹配到合适的子路由。如果你想要渲染点什么，可以提供一个 **空的 子路由**：

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

html:

```html
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <p>
    <router-link to="/user/foo">/user/foo</router-link>
    <router-link to="/user/foo/profile">/user/foo/profile</router-link>
    <router-link to="/user/foo/posts">/user/foo/posts</router-link>
  </p>
  <router-view></router-view>
</div>
```

JS:

```js
const User = {
  template: `
    <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view>
    </div>
  `
}

const UserHome = { template: '<div>Home</div>' }
const UserProfile = { template: '<div>Profile</div>' }
const UserPosts = { template: '<div>Posts</div>' }

const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        // UserHome will be rendered inside User's <router-view>
        // when /user/:id is matched
        { path: '', component: UserHome },
				
        // UserProfile will be rendered inside User's <router-view>
        // when /user/:id/profile is matched
        { path: 'profile', component: UserProfile },

        // UserPosts will be rendered inside User's <router-view>
        // when /user/:id/posts is matched
        { path: 'posts', component: UserPosts }
      ]
    }
  ]
})

const app = new Vue({ router }).$mount('#app')
```

效果：

![嵌套路由](https://i.loli.net/2021/02/04/94NCwM7kYObtXfo.gif)