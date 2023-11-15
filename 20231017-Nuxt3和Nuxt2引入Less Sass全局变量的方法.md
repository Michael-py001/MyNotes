# 20231017-Nuxt3和Nuxt2引入Less Sass全局变量的方法

网上的教程鱼龙混杂，Nuxt3比较新，大多数文章还是能直接给出答案，但是想使用Nuxt2时，找到的教程都不能直接给出正确答案，就算是步骤操作是对的，但是下载下来的包版本不支持，这也是个坑。

以下就分享下我配置Less全局变量的教程(Sass同样适用)：

## Nuxt3

### 通过Vite配置

首先需要安装less和less-loader

```bash
pnpm install less less-loader

//当前实测可用版本
"nuxt": "^3.7.4"
"less": "^4.2.0",
"less-loader": "^11.1.3",
```

> [Vite - Nuxt Configuration Reference · Nuxt](https://nuxt.com/docs/api/configuration/nuxt-config#vite)

```js
// nuxt.config.ts
vite: {
    css: {
      preprocessorOptions: {
        less: {
          additionalData: '@import "@/assets/css/variable.less";',
        },
      },
    },
  },
```

### 通过官方插件实现

> [nuxt-community/style-resources-module: Style Resources for Nuxt 3 (github.com)](https://github.com/nuxt-community/style-resources-module)

同样要安装less loader

```bash
yarn add less-loader less @nuxtjs/style-resources
```

```js
//nuxt.config.js
export default {
  buildModules: [
    '@nuxtjs/style-resources',
  ],

  styleResources: {
   // your settings here
   sass: [],
   scss: [],
   less: [],
   stylus: [],
   hoistUseStatements: true  // Hoists the "@use" imports. Applies only to "sass", "scss" and "less". Default: false.
  }
}
```

例子：

```js
// nuxt.config.js
export default {
  css: ['~assets/global.less'],
  buildModules: ['@nuxtjs/style-resources'],
  styleResources: {
    less: './assets/vars/*.less'
  }
}
```

 `assets/global.less`

```css
h1 {
  color: @green;
}
```

`assets/vars/variables.less`

```
@gray: #333;
```

`assets/vars/more_variables.less`

```
@green: #00ff00;
```

`pages/index.vue`

```vue
<template>
  <div>
    <!-- This h1 will be green -->
    <h1>Test</h1>
    <test/>
  </div>
</template>

<script>
import Test from '~/components/Test'

export default {
  components: { Test }
}
</script>
```

`components/Test.vue`

```vue
<template>
  <div class="ymca">
    Test
  </div>
</template>

<style lang="less">
  .ymca {
    color: @gray; // will be resolved to #333
  }
</style>
```

## Nuxt2

目前找到的方法只有一种：使用官方插件。

> [nuxt-community/style-resources-module at v1 (github.com)](https://github.com/nuxt-community/style-resources-module/tree/v1)

注意！需要安装v1版本的@nuxtjs/style-resources，不然在Nuxt2会报错！

```
yarn add -D @nuxtjs/style-resources@1
```



- SASS: `yarn add sass-loader sass` (for Nuxt 2 use: `sass-loader@10`)
- LESS: `yarn add less-loader less`
- Stylus: `yarn add stylus-loader stylus`

```js
export default {
 styleResources: {
    less: ['./assets/css/variable.less'],
  },
  css: ['./assets/css/variable.less','swiper/dist/css/swiper.css'],
  buildModules: [
    '@nuxtjs/style-resources'
  ],
}
```

