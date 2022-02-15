[TOC]



# Vue中如何使用hightlight.js代码高亮

> 参考资料：
>
> https://www.metachris.com/2017/02/vuejs-syntax-highlighting-with-highlightjs/
>
> https://www.jianshu.com/p/d42e21d17ab0
>
> https://github.com/metachris/vue-highlightjs/blob/master/index.js

## 安装

```
npm install highlight.js 
// or
yarn add highlight.js
```

## 组件内使用

### 引入js css文件

```js
import hljs from "highlight.js";
// 样式文件
import("highlight.js/styles/atom-one-dark.css");
```

### 自定义指令

```js
// 自定义插件
  directives: {
    // 自定义指令 v-highlight
    highlightjs: {
      // 英文博客的内容：https://www.metachris.com/2017/02/vuejs-syntax-highlighting-with-highlightjs/
      // 只调用一次，指令第一次绑定到元素时调用
      bind: function (el, binding) {
        let targets = el.querySelectorAll("code");
        targets.forEach((target) => {
          if (typeof binding.value === "string") {
            target.textContent = binding.value;
          }
          hljs.highlightBlock(target);
        });
      },
      // 指令所在组件的 VNode 及其子 VNode 全部更新后调用。
      componentUpdated: function (el, binding) {
        let targets = el.querySelectorAll("code");
        targets.forEach((target) => {
          console.log(target);
          if (typeof binding.value === "string") {
            // if (binding.value) {//因为Boolean("空字符串")的结果的false，所以会导致下面的if在删除到最后一个字符时，结果不成立，内容不会更新，所以要改成if (typeof binding.value === "string")
            target.textContent = binding.value;
            hljs.highlightBlock(target);
          }
        });
      },
```

### 遇到的坑

1. 网上查找到的文章多多少少都有些错误的地方，只有自己踩过才知道。比如highlight渲染高亮不能实时更新，只渲染页面加载的第一次，后续如果需要编辑代码实时更新，则不起效了。

   比如网上查找到的代码：

   ```html
   <template>
           <pre v-highlightjs>
             <code class="json">{{value}}</code>
           </pre>
   <template>
   ```

   这里使用了自定义指令`v-highlightjs`，但是这样写只能渲染第一次，没有发挥自定义指令的最大化效果。

   最佳代码：

   ```html
   <template>
           <pre v-highlightjs=“value”>
             <code class="json"></code>
           </pre>
   <template>
   ```

   修改后，直接在指令上传入代码：`v-highlightjs=“value”`，就可以实时更新了。

2. 如果要删除代码，但是会发现删除到最后一个字符会删不掉，这是为啥？

   > https://www.metachris.com/2017/02/vuejs-syntax-highlighting-with-highlightjs 代码出处

   ```js
   componentUpdated: function (el, binding) {
       // after an update, re-fill the content and then highlight
       let targets = el.querySelectorAll('code')
       targets.forEach((target) => {
         if (binding.value) {
           target.textContent = binding.value
           hljs.highlightBlock(target)
         }
       })
     }
   ```

   上面使用了` if (binding.value)`判断（有的文章甚至没有写判断，直接循环），但是删除到最后的一个字符会删不掉，这是因为删到最后一个字符时，binding.value为假，判断就不成立了，里面的代码自然不会执行。

   优化后的代码：

   ```js
    componentUpdated: function (el, binding) {
           let targets = el.querySelectorAll("code");
           targets.forEach((target) => {
             if (typeof binding.value === "string") {
               // if (binding.value) {//因为Boolean("空字符串")的结果的false，所以会导致下面的if在删除到最后一个字符时，结果不成立，内容不会更新，所以要改成if (typeof binding.value === "string")
               target.textContent = binding.value;
             }
             hljs.highlightBlock(target);
           });
         }
   ```

   

## 全局使用

### 封装成vue插件

[官方文档---自定义插件](https://links.jianshu.com/go?to=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fguide%2Fplugins.html)

[官方文档---自定义指令](https://links.jianshu.com/go?to=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fguide%2Fcustom-directive.html)

新建`highlight.js`文件，并添加：

```js
// src/utils/highlight.js 文件路径，纯属自定义

// highlight.js  代码高亮指令
import Hljs from 'highlight.js';
import 'highlight.js/styles/tomorrow-night.css'; // 代码高亮风格，选择更多风格需导入 node_modules/hightlight.js/styles/ 目录下其它css文件

let Highlight = {};
// 自定义插件
Highlight.install = function (Vue) {
    // 自定义指令 v-highlight
    Vue.directive('highlight', {
        // 被绑定元素插入父节点时调用
        bind: function (el, binding) {
        let targets = el.querySelectorAll("code");
        targets.forEach((target) => {
          if (binding.value) {
            target.textContent = binding.value;
          }
          hljs.highlightBlock(target);
        });
      },
      // 指令所在组件的 VNode 及其子 VNode 全部更新后调用。
      componentUpdated: function (el, binding) {
        let targets = el.querySelectorAll("code");
        targets.forEach((target) => {
          if (typeof binding.value === "string") {
            // if (binding.value) {//因为Boolean("空字符串")的结果的false，所以会导致下面的if在删除到最后一个字符时，结果不成立，内容不会更新，所以要改成if (typeof binding.value === "string")
            target.textContent = binding.value;
          }
          hljs.highlightBlock(target);
        });
      },
    })
};

export default Highlight;
```

### 3. 全局引入自定义插件

在`src/main.js`中：

```js
// highlight.js代码高亮插件
import Highlight from './utils/highlight'; // from 路径是highlight.js的路径，纯属自定义
Vue.use(Highlight);
```

### 4. 使用

```html
<div id="codeView" v-highlight=“code”>
    <pre><code></code></pre>
</div>
```

更换更多的代码高亮风格，需要切换`highlight.js`中导入的`css`文件。

>
> 参考文章链接：https://www.jianshu.com/p/e35f6d62b3bd
>