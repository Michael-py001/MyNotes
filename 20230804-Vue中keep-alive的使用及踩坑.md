# 20230804-Vue中keep-alive的使用及踩坑

## 需求场景

从A页面点击"资产登记"按钮，跳转到B页面，添加资产后返回A页面，同时要保持A页面表单输入的数据。

## 需求分析

这种需要页面数据留存的场景有两种方案：

1. **手动缓存**：跳转前缓存A页面的表单数据，从B切换回A时从缓存中赋值给表单，从而实现数据留存。
   **缺点**：需要缓存的数据较多，比如A页面是多个Tab切换的，需要把Tab的激活下标也缓存，还要维护缓存的数据：包括进入页面和退出页面时的缓存清除，防止数据重复赋值。
2. **使用`keep-alive`实现页面缓存**：无需手动做数据缓存，`keep-alive`组件自带这个能力，需要做的是在`onActivated、onDeactivated`两个生命周期中看实际需要做数据重置。

## 实现方案

### 配置keep-alive组件

视图模板中对`keep-alive`做支持，根据路由表中`keepAlive`字段判断是否启用keep-alive组件

```vue
<a-layout-content class="content">
  <router-view  v-slot="{ Component, route }">
      <keep-alive v-if="$route.meta.keepAlive">
        <component :is="Component" :key="$route.fullPath" />
      </keep-alive>
      <component :is="Component" v-if="!$route.meta.keepAlive" :key="$route.fullPath"/>
  </router-view>
</a-layout-content>
```

路由表中配置`keepAlive`能力：

> 需要注意的是，这里需要配置两个页面开启keep-alive，因为经过实践验证，只在A页面开启，B页面不开启时，从B页面跳转回A后，不能触发A页面的keep-alive能力，数据会被重置，相当于跳转到普通页面。

```js
// A页面启用keepalive  
{
    path: '/cloudVote',
    name: 'cloudVote',
    component: () => import('@/views/cloudVote/Index/index.vue'),
    meta: {
      title: '云表决',
      keepAlive: true,
    },
  },
```

```js
//B页面也要启用keepalive
{
    path: '/asset/assetChooseTypes',
    name: 'assetChooseTypes',
    component: () => import('@/views/asset/assetChooseTypes'),
    meta: {
        title: '资产登记-选择资产类型',
        keepAlive: true,
    },
},
```

### 实现效果

![11](https://s2.loli.net/2023/08/07/efMbTQHp3BCRKIG.gif)

### 需要keep-alive的数据重置问题

如果从X页面跳转到A页面，且跳转的路由参数完全一致，这时如果你之前在A页面填了数据，这时A页面的数据如果没有手动重置表单数据，那么数据还会存在。

当然你会想到在页面进入时在`onActivated`做数据重置不就行了，但是这种方法你也要考虑到从B跳转回A时也会触发这个声明周期。那么还要从路由参数中做标记判断是否重置数据，这样无疑是增加代码量和维护难度的。

#### 小技巧

这里推荐一个小技巧，通过设置不同路由参数，使keep-alive不能生效，达到数据自动重置的效果。

这里通过传递时间戳，达到每次路由跳转的参数都不一致的效果，进入到keep-alive页面时会重新触发`onMounted`、`onActivated`的声明周期，页面的数据会被重置，这样就无需手动重置了。

```js
  router.push({
    path: _path,
    query: {
      ..._query,
      timeSmap: new Date().getTime(),//每次跳转，传递不同的参数值
    },
```

