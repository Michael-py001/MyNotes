# 20220630-uniapp Vue3项目如何传入插件@vitejs/plugin-vue的vue()的参数？如何开启响应式语法糖？

> [响应性语法糖 | Vue.js (vuejs.org)](https://staging-cn.vuejs.org/guide/extras/reactivity-transform.html#explicit-opt-in)

以上的官方文档写到，要显式开启**响应性语法糖**，要在`vite.config.js`下配置`reactivityTransform: true`，

那么问题来了，我的uniapp项目是使用Hbuilder X创建的，连这个`vite.config.js`文件都没有，要怎么传这个参数进去呢？

## 创建vite.config.js文件

​	Hbuilder X创建的项目默认是没有vite.config.js文件的，需要我们手动创建；

在根目录创建该文件，准备下一步。

## 通过查看uniapp的编译源码找到答案

我首先去找到使用cli方式创建的Vite项目是如何配置vite.config.js文件的；

```js
//cli方式创建的Vite项目的vite.config.js文件
import { defineConfig } from 'vite'
import uni from '@dcloudio/vite-plugin-uni'
// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    uni(),
  ],
})

```

再来看下使用Vue3官方模板创建的Vite项目配置

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()]
})
```

对比可以看出，uniapp项目需要用他自己的插件进行编译

- 这里需要注意，如果你只创建了vite.config.js文件但是没用填入任何配置，直接运行是会报错的
- 使用上面Vue3官方模板的配置也是会报错的

所以我就想到了直接查看这个@dcloudio/vite-plugin-uni的源码，看看里面到底有没有提供这个给vue(@vitejs/plugin-vue)传入参数的功能。

![image-20220630120125275](https://s2.loli.net/2022/06/30/kbi4QlGKua9NnvW.png)

![image-20220630134034900](https://s2.loli.net/2022/06/30/nvkXUVCRP6dILQe.png)

实际调用的地方：

```js
plugins.unshift((0, vue_1.createPluginVueInstance)((0, vue_1.initPluginVueOptions)(options, uniPlugins, uniPluginOptions)));
```

可以看到这里调用了`vue_1.createPluginVueInstance`这个方法，它的参数是调用`initPluginVueOptions`这个方法处理过的，看看这个方法是干嘛的：

```js
function createPluginVueInstance(options) {
    delete require.cache[pluginVuePath];
    delete require.cache[normalizedPluginVuePath];
    const vuePlugin = require('@vitejs/plugin-vue');
    return vuePlugin(options);
}
```

可以看到，实际上源码也是使用了`@vitejs/plugin-vue`，只不过调用的方式和处理参数包了一层。

可以从源码看到：

这个`vueOptions`被提取了出来，整个`initPluginVueOptions`就是专门处理这个参数的，返回的内容被传进去了上面说的`createPluginVueInstance`方法，这里就是源码的核心内容。

```js
const vueOptions = options.vueOptions || (options.vueOptions = {});
```

![image-20220630114028360](https://s2.loli.net/2022/06/30/uTGLejlaHCpJ47i.png)

在结合插件的主入口向外暴露的方法：

```js
function uniPlugin(rawOptions = {}){
	...
}
```

这里的`rawOptions`就是我们传入的参数

![image-20220630135953196](https://s2.loli.net/2022/06/30/DPjOpN47nXLcq6V.png)

所以按照上面的处理逻辑，需要传入`{vueOptions:{...}}`这个选项，就能传给`@vitejs/plugin-vue`这个插件作为参数了。

在下面还看到了处理JSX参数的`initPluginVueJsxOptions`方法，也是大同小异。

在plugin.js的主调用方法`initExtraPlugins`里面输出options，就能看到我们传进去的和内部自带的参数了：

![image-20220630135607385](https://s2.loli.net/2022/06/30/FJEeWLjU9Yc64CV.png)



那么开启显式启用响应性语法糖就很简单了:

```js
//vite.config.js
import { defineConfig } from "vite";
import uni from "@dcloudio/vite-plugin-uni";

export default defineConfig({
  plugins: [uni({
    vueOptions:{
      reactivityTransform: true
    }
  })],
});
```

![image-20220630114422564](https://s2.loli.net/2022/06/30/GAXh2Uifrtm8sxp.png)

不过开启后会有警告：

说这是实验性功能，在后面的版本可能会有修改，

![image-20220630141036266](https://s2.loli.net/2022/06/30/qwCT6m9UnDRXhYL.png)