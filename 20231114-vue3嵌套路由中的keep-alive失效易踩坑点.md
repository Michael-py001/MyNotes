# 20231114-vue3嵌套路由中的keep-alive失效易踩坑点



在`layout`中已经配置了keep-alive：

```html
<router-view v-slot="{ Component, route }">
    <keep-alive v-if="$route.meta.keepAlive">
        <component :is="Component" :key="$route.fullPath" />
    </keep-alive>
    <component
        :is="Component"
        v-if="!$route.meta.keepAlive"
        :key="$route.fullPath"
    />
</router-view>
```

一级路由配置：

```js
import Layout from '@/layout/index.vue'
import systemManage from './modules/systemManage'

export const permRoutes = [
    {
        path: '/',
        component: () => import('@/views/home/index.vue'),
        meta: { showMenu: false },
    },
    systemManage,
]

export const constantRoutes = [
    {
        path: '/login',
        component: () => import('@/views/login/index.vue'),
    },
    {
        path: '/',
        component: Layout,
        children: permRoutes,
    },
]
```

其中一个页面中的嵌套路由：

```js
export default {
    path: 'systemManage',
    component: () => import('@/views/systemManage/index/index.vue'),
    meta: { name: '系统维护', keepAlive: true },
    children: [
        {
            path: 'menus',
            component: () => import('@/views/systemManage/menus/index.vue'),
            meta: { name: '菜单管理' },
        }
      ...
     ]
   }
```

## 易错点

因为我们使用了meta.keepAlive判断是否启用keep-alive，所以一般我们也会在嵌套路由中配置，但是在嵌套路由有一个特殊点：

- 子路由中设置了meta.keepAlive，但是其父路由没有设置，则整个子路由都不生效。
- 其父路由设置了meta.keepAlive，整个子路由的meta都会继承父路由的meta,相当于使用了Object.assign合并属性

也就是说在上面的配置中，`菜单管理`页面输出当前路由信息，meta里面也会包含`系统维护`中的meta信息`keepAlive:true`。

容易出错的点就是没有在`系统维护`中设置keepAlive，导致他的子路由即使设置了keepAlive:true也不生效。

