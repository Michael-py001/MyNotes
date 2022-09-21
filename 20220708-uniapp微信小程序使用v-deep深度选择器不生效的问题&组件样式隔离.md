# 20220708-uniapp微信小程序使用v-deep深度选择器不生效的问题&组件样式隔离

如果你想改第三方的组件库，需要用到深度选择器，可以使用Vue3提供的v-deep，但是使用uniapp编译到h5和微信小程序时你会发现在小程序上v-deep有些页面生效，有些页面却不生效了，这到底是为啥？为什么说是有些而不是全部呢？

先来看代码

父组件：

```html
<template>
	<view class="content">
		<Item></Item>
	</view>
</template>

<script setup>
  import Item from './components/item'
</script>
```

子组件：

```html
<template>
  <view class="flex-row equal-division">
    <Child src="/static/logo.png" v-for="(item,index) in 4" :key="index"></Child>
  </view>
</template>

<script setup>
  import Child from './Child'
</script>
<style lang="scss">
  .equal-division {
       margin-top: 22rpx;
       ::v-deep .my-image {
         margin-right: 27rpx;
       }
     }
</style>
```

孙组件：

```html
<template>
  <image class="my-image" :src="src" mode="aspectFill"></image>
</template>


<script setup>
  const props = defineProps({
    src:String
  })
</script>
<style lang="scss" scoped>
  .my-image {
    width: 100rpx;
    height: 100rpx;
  }
</style>
```

上面的代码如果你编译到小程序，那么给`my-image`设置的maigin-right是不会生效的。

## 解决方案

1. 把`v-deep`的样式移动到父页面即可

2. 在子组件加上下面的配置:

   ```js
   options: {
       styleIsolation: 'isolated'
     }
   ```

## 问题出现的原因

这是因为微信小程序的**组件样式隔离**导致的。

> [组件模板和样式 | 微信开放文档 (qq.com)](https://developers.weixin.qq.com/miniprogram/dev/framework/custom-component/wxml-wxss.html#组件样式隔离)

默认情况下，自定义组件的样式只受到自定义组件 wxss 的影响。除非以下两种情况：

- `app.wxss` 或页面的 `wxss` 中使用了标签名选择器（或一些其他特殊选择器）来直接指定样式，这些选择器会影响到页面和全部组件。通常情况下这是不推荐的做法。
- 指定特殊的样式隔离选项 `styleIsolation` 。

```js
Component({
  options: {
    styleIsolation: 'isolated'
  }
})
```

`styleIsolation` 选项从基础库版本 [2.6.5](https://developers.weixin.qq.com/miniprogram/dev/framework/compatibility.html) 开始支持。它支持以下取值：

- `isolated` 表示启用样式隔离，在自定义组件内外，使用 class 指定的样式将不会相互影响（一般情况下的默认值）；
- `apply-shared` 表示页面 wxss 样式将影响到自定义组件，但自定义组件 wxss 中指定的样式不会影响页面；
- `shared` 表示页面 wxss 样式将影响到自定义组件，自定义组件 wxss 中指定的样式也会影响页面和其他设置了 `apply-shared` 或 `shared` 的自定义组件。（这个选项在插件中不可用。）

从小程序基础库版本 [2.10.1](https://developers.weixin.qq.com/miniprogram/dev/framework/compatibility.html) 开始，也可以在页面或自定义组件的 json 文件中配置 `styleIsolation` （这样就不需在 js 文件的 `options` 中再配置）。例如：

```json
{
  "styleIsolation": "isolated"
}
```

## Vue3里如何加上这个options?

如果直接使用`script setup`是不行的，因为uniapp的`@dcloudio/uni-app`并没有导出这个`options`方法。

但是你可以同时使用`<script setup>`和`<script>`标签

```html
<script >
  export default {
     options: {
        styleIsolation: 'shared'
      }
  }
</script>

<script setup>
  import Child from './Child'
</script>

<style lang="scss">
  .equal-division {
       margin-top: 22rpx;
       ::v-deep .my-image {
         margin-right: 27rpx;
       }
     }
</style>

```

注意的是，像上面的代码，你需要在写v-deep的子组件上添加options，而不是在孙组件里。

至于开头说的为什么的有些情况正常呢？是因为你可能并没有使用三个组件，而是直接是父子组件，那么在父组件的页面些v-deep是可以直接生效的。😂