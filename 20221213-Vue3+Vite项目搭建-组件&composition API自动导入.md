# Vue3项目搭建-组件&composition API自动导入

### 使用到的插件：

- unplugin-auto-import
- unplugin-vue-components

## unplugin-auto-import介绍

> 官方github地址：[antfu/unplugin-auto-import: Auto import APIs on-demand for Vite, Webpack and Rollup (github.com)](https://github.com/antfu/unplugin-auto-import)

是一款自动按需引入APIs的工具，支持Vite, Webpack, Rollup and esbuild，且具有完备的TS支持。

### 使用示例

without

```js
import { computed, ref } from 'vue'
const count = ref(0)
const doubled = computed(() => count.value * 2)
```

with

```js
const count = ref(0)
const doubled = computed(() => count.value * 2)
```

------

without

```js
import { useState } from 'react'
export function Counter() {
  const [count, setCount] = useState(0)
  return <div>{ count }</div>
}
```

with

```js
export function Counter() {
  const [count, setCount] = useState(0)
  return <div>{ count }</div>
}
```

### 安装

```bash
npm i -D unplugin-auto-import
```

### 配置

#### Vite 

```js
// vite.config.ts
import AutoImport from 'unplugin-auto-import/vite'

export default defineConfig({
  plugins: [
    AutoImport({ /* options */ }),
  ],
})
```

更多配置请看[官方文档](https://github.com/antfu/unplugin-auto-import)

## unplugin-vue-components介绍

> [官方gitgub地址](https://github.com/antfu/unplugin-vue-components#readme)

专门用于Vue的按需加载自动引入组件的插件。

### 安装

```bash
npm i unplugin-vue-components -D
```

### Vite示例

```js
// vite.config.ts
import Components from 'unplugin-vue-components/vite'

export default defineConfig({
  plugins: [
    Components({ /* options */ }),
  ],
})
```

### 引入指定路径下的组件

插件会自动引入`src/components`下的组件(公共组件)，可以在配置中的`dirs`修改

It will automatically turn this

```vue
<template>
  <div>
    <HelloWorld msg="Hello Vue 3.0 + Vite" />
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>
```

into this

```vue
<template>
  <div>
    <HelloWorld msg="Hello Vue 3.0 + Vite" />
  </div>
</template>

<script>
import HelloWorld from './src/components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>
```

### 引入UI组件库

插件内置了主流的UI组件库的resolvers。

使用方法：

```js
// vite.config.js
import Components from 'unplugin-vue-components/vite'
import {
  AntDesignVueResolver,
  ElementPlusResolver,
  VantResolver,
} from 'unplugin-vue-components/resolvers'

// your plugin installation
Components({
  resolvers: [
    AntDesignVueResolver(),
    ElementPlusResolver(),
    VantResolver(),
  ],
})
```

或者你可以自定义自己的resolver：

```js
Components({
  resolvers: [
    // example of importing Vant
    (componentName) => {
      // where `componentName` is always CapitalCase
      if (componentName.startsWith('Van'))
        return { name: componentName.slice(3), from: 'vant' }
    },
  ],
})
```

## 实际使用案例配置

```js
// vite.config.ts
import AutoImport from 'unplugin-auto-import/vite'
import { AntDesignVueResolver, ElementPlusResolver } from 'unplugin-vue-components/resolvers';
import Icons from 'unplugin-icons/vite';
import IconsResolver from 'unplugin-icons/resolver';
export default defineConfig({
  plugins: [
    AutoImport({
      //自动引入vue、vue-router、vuex的API，比如：ref, reactive, toRef 等，不需要每一个组件都引入一遍
      imports: [
      'vue',
      'vue-router',
      {
        vuex: ['useStore'],
      },
    ],
    resolvers: [
        // 自动导入 Element Plus 相关函数，如：ElMessage, ElMessageBox...
        ElementPlusResolver(),
        //自动导入AntDesign相关函数，使用了那个组件库就用哪个
        AntDesignVueResolver()
        // 自动导入图标组件
        IconsResolver({
          prefix: 'Icon',
        }),
      ],
    // Filepath to generate corresponding .d.ts file.
    // 默认生成在根目录：./auto-imports.d.ts， 只在typescript安装才生效.
    // 设置false关闭.
    dts: false,
    // 生成文件，包含所引入的所有 API 名称，防止eslint报错
    eslintrc: {
      enabled: false, // 每次都会生成文件，如果已存在文件设置默认 false，需要更新时再打开，防止每次更新都重新生成
      filepath: './.eslintrc-auto-import.json', // 生成文件地址和名称
      globalsPropValue: true,
    },
   }),
  //自动引入组件
  Components({
      // 是否生成components.d.ts类型声明文件,第一次生成后关闭，需要更新时
      // also accepts a path for custom filename 可以自定义文件路径名称
      // default: `true` if package typescript is installed ，如果ts开启默认是true
      dts: false,
      directoryAsNamespace: true,
      resolvers: [AntDesignVueResolver()],
    }),
 ]
})
```

