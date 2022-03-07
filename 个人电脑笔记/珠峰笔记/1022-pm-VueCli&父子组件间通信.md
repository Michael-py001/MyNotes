# Vue CLI & 父子组件通信

[TOC]



## Vue CLI介绍

Vue CLI 是一个基于 Vue.js 进行快速开发的完整系统，提供：

- 通过 `@vue/cli` 实现的交互式的项目脚手架。
- 通过 `@vue/cli` + `@vue/cli-service-global` 实现的零配置原型开发。
- 一个运行时依赖 (`@vue/cli-service`)，该依赖：
  - 可升级；
  - 基于 webpack 构建，并带有合理的默认配置；
  - 可以通过项目内的配置文件进行配置；
  - 可以通过插件进行扩展。
- 一个丰富的官方插件集合，集成了前端生态中最好的工具。
- 一套完全图形化的创建和管理 Vue.js 项目的用户界面。

## 该系统的组件

###  CLI

​		CLI (`@vue/cli`) 是一个全局安装的 npm 包，提供了终端里的 `vue` 命令。它可以通过 `vue create` 快速搭建一个新项目，或者直接通过 `vue serve` 构建新想法的原型。你也可以通过 `vue ui` 通过一套图形化界面管理你的所有项目。

###  CLI 服务

​		CLI 服务 (`@vue/cli-service`) 是一个开发环境依赖。它是一个 npm 包，局部安装在每个 `@vue/cli` 创建的项目中。

CLI 服务是构建于 [webpack](http://webpack.js.org/) 和 [webpack-dev-server](https://github.com/webpack/webpack-dev-server) 之上的。它包含了：

- 加载其它 CLI 插件的核心服务；
- 一个针对绝大部分应用优化过的内部的 webpack 配置；
- 项目内部的 `vue-cli-service` 命令，提供 `serve`、`build` 和 `inspect` 命令。

### CLI 插件

​		CLI 插件是向你的 Vue 项目提供可选功能的 npm 包，例如 Babel/TypeScript 转译、ESLint 集成、单元测试和 end-to-end 测试等。Vue CLI 插件的名字以 `@vue/cli-plugin-` (内建插件) 或 `vue-cli-plugin-` (社区插件) 开头，非常容易使用。

当你在项目内部运行 `vue-cli-service` 命令时，它会自动解析并加载 `package.json` 中列出的所有 CLI 插件。

插件可以作为项目创建过程的一部分，或在后期加入到项目中。它们也可以被归成一组可复用的 preset。

### 安装vue CLI

可以使用下列任一命令安装这个新的包：

```bash
npm install -g @vue/cli
# OR
yarn global add @vue/cli
```

安装之后，你就可以在命令行中访问 `vue` 命令。你可以通过简单运行 `vue`，看看是否展示出了一份所有可用命令的帮助信息，来验证它是否安装成功。

你还可以用这个命令来检查其版本是否正确：

```bash
vue --version
```

#### 升级

如需升级全局的 Vue CLI 包，请运行：

```bash
npm update -g @vue/cli

# 或者
yarn global upgrade --latest @vue/cli
```

#### 项目依赖

上面列出来的命令是用于升级全局的 Vue CLI。如需升级项目中的 Vue CLI 相关模块（以 `@vue/cli-plugin-` 或 `vue-cli-plugin-` 开头），请在项目目录下运行 `vue upgrade`：

```text
用法： upgrade [options] [plugin-name]

（试用）升级 Vue CLI 服务及插件

选项：
  -t, --to <version>    升级 <plugin-name> 到指定的版本
  -f, --from <version>  跳过本地版本检测，默认插件是从此处指定的版本升级上来
  -r, --registry <url>  使用指定的 registry 地址安装依赖
  --all                 升级所有的插件
  --next                检查插件新版本时，包括 alpha/beta/rc 版本在内
  -h, --help            输出帮助内容
```

### 快速原型开发

你可以使用 `vue serve` 和 `vue build` 命令(前提是安装了vue CLI)对单个 `*.vue` 文件进行快速原型开发，不过这需要先额外安装一个全局的扩展：

```bash
npm install -g @vue/cli-service-global
```

`vue serve` 的缺点就是它需要安装全局依赖，这使得它在不同机器上的一致性不能得到保证。因此这只适用于快速原型开发。

### 开发环境使用—vue serve

**vue serve**：在开发环境模式下零配置为 .js 或 .vue 文件启动一个服务器

```bash
Usage: serve [options] [entry]

Options:
  -o, --open  打开浏览器
  -c, --copy  将本地 URL 复制到剪切板
  -h, --help  输出用法信息
```

你所需要的仅仅是一个 `App.vue` 文件：

```vue
<template>
  <h1>Hello!</h1>
</template>
```

然后在这个 `App.vue` 文件所在的目录下运行：

```bash
vue serve
```

`vue serve` 使用了和 `vue create` 创建的项目相同的默认设置 (webpack、Babel、PostCSS 和 ESLint)。它会在当前目录自动推导入口文件——入口可以是 `main.js`、`index.js`、`App.vue` 或 `app.vue` 中的一个。你也可以显式地指定入口文件：

```bash
vue serve MyComponent.vue
```

如果需要，你还可以提供一个 `index.html`、`package.json`、安装并使用本地依赖、甚至通过相应的配置文件配置 Babel、PostCSS 和 ESLint。

### 生产环境使用—vue build

**vue build**：在生产环境模式下零配置构建一个 .js 或 .vue 文件

```bash
Usage: build [options] [entry]


Options:

  -t, --target <target>  构建目标 (app | lib | wc | wc-async, 默认值：app)
  -n, --name <name>      库的名字或 Web Components 组件的名字 (默认值：入口文件名)
  -d, --dest <dir>       输出目录 (默认值：dist)
  -h, --help             输出用法信息
```

你也可以使用 `vue build` 将目标文件构建成一个生产环境的包并用来部署：

```bash
vue build MyComponent.vue
```

`vue build` 也提供了将组件构建成为一个库或一个 Web Components 组件的能力。查阅[构建目标](https://cli.vuejs.org/zh/guide/build-targets.html)了解更多。

### .vue文件包含的部分

- `<template>`：模板的内容，相当于局部组件中`template:#hello`的内容
- `<script>`：JS的内容，通过`export default`导出的一个对象，相当于局部组件中`let hello = {...}`的内容。
- `<style>`：样式内容

```vue
/* 模板内容 */
<template>
    <div>msg:{{msg}}</div>
</template>

/* JS内容 */
<script>
export default {
    // data是一个函数
    data(){
        return {
            msg:10
        }
    }
}
</script>

/* 样式内容 */
<style>
    div {
        color: aquamarine;
    }
</style>
```

## 组件间的通信

创建目录结构：

```
-/components
	|-GrandSon.vue
	|-Parent.vue
	|-Son.vue
-App.vue
```

组件的使用步骤：

1. 定义+引入
2. 注册组件
3. 使用组件

`App.vue`文件引入其他组件：

```js
/* 模板内容 */
<template>
    <div>
        <Parent></Parent>
    </div>
</template>

/* JS内容 */
<script>
//1. 定义+引入 2. 注册组件 3. 使用组件
import Parent from "./components/Parent"
export default {
    // data是一个函数
    data(){
        return {
            age:10
        }
    },
    // 注册组件
    components:{
        Parent
    }
    
}
</script>

/* 样式内容 */
<style>
    div {
        color: aquamarine;
    }
</style>
```

组件中获取数据主要有两种方式：

1. 通过`data`函数/选项里设置的数据
2. 通过`props`

`Parent.vue`文件：

```js
/* 模板内容 */
<template>
    <div>I am:{{msg}}</div>
</template>

/* JS内容 */
<script>
// 通过props data获取数据
export default {
    // data是一个函数
    data(){
        return {
            msg:"parent"
        }
    }
}
</script>

/* 样式内容 */
<style>
    div {
        color: aquamarine;
    }
</style>
```

### 父组件->子组件通信

> 父组件传给子组件的通信，通过`props`传递数据。
>
> - 父组件上引入子组件，在子组件标签上设置prop属性
> - 在子组件JS里设置props选项，填入设置的prop属性。

> 需求：把`Parent.vue`中data中的msg内容传递给`Son.vue`子组件。

1. 首先引入子组件

   ```js
   import Son from "./Son";
   ```

2. 注册组件

   ```js
   components:{
           Son
       }
   ```

3. 使用组件

   ```js
   <Son :myName="msg"></Son>
   ```

   注意：这里如果只写`myName="msg"`的意思是，把字符串`msg`传给了子组件`Son`，加上`:`表示vue中的动态属性，里面的`msg`表示一个变量。

`Parent.vue`：
    通过**动态属性**`:myName="msg"`把msg变量的值传给了Son组件。

```vue
/* 模板内容 */
<template>
    <div class="parent">I am:{{msg}}
        <Son :myName="msg"></Son>
    </div>
</template>

/* JS内容 */
<script>
// 引入子组件.
import Son from "./Son";
// 通过props data获取数据
export default {
    // data是一个函数
    data(){
        return {
            msg:"parent"
        }
    },
    // 注册组件
    components:{
        Son
    }
}
</script>

/* 样式内容 */
<style>
    .parent {
        color: aquamarine;
        border: 5px solid rgb(214, 104, 30);
    }
</style>
```

`Son.vue`:

   在子组件中，通过`props`选项，接收从父组件中给Son标签设置属性(prop)传过来的`myName`值。

```vue
/* 模板内容 */
<template>
    <div class="child">I am:{{msg}}
        <div>通过props[属性]从父组件传过来的myName值：{{myName}}</div>
    </div>
</template>

/* JS内容 */
<script>
export default {
    props:['myName'],
    // data是一个函数
    data(){
        return {
            msg:"son"
        }
    }
}
</script>

/* 样式内容 */
<style>
    .child {
        margin-top: 10px;
         color: rgb(60, 219, 39);
        border: 3px solid rgb(218, 233, 17);
    }
</style>
```

运行`vue serve`预览效果：

结构总结：

- 在根组件`App.vue`中引入了子组件`Parent.vue`
- 然后又在`Parent.vue`中，引入了子组件`Son.vue`

所以页面的内容是`Parent`里嵌套了子组件`Son`的内容。

![image-20201024004938615](https://i.loli.net/2020/10/24/A9Mvl47qbfLtoHV.png)

#### 对props的值进行校验

> 如果要对传过来的值进行校验判断，则写成对象形式。

- `type:[String,Number]`, //设置值的数据类型
- `default`:"I am default value", //设置默认值
- `require:true,`   //设置必须传值,不能与default同时存在
- `validator`  :func(){...}    //对传过来的值进行自己的规则

```js
 // props:['myName'],
    // 如果要对传过来的值进行校验判断，则写成对象形式
    props:{
        myName:{
            type:[String,Number], //设置值的数据类型
            default:"I am default value", //设置默认值
            // require:true,  //设置必须传值,不能与default同时存在
            validator:(value)=>{  //对传过来的值进行自己的规则校验
                return value.length>=5
            }
        }
    },
```

如果在父组件中，不传子组件props上设置的属性，则会触发`default`

```html
- <Son :myName="msg"></Son>
+ <Son :myName="msg"></Son>
```

### 子组件->父组件通信

> 子组件把信息传给父组件，是通过事件的方式-->`$emit()`。

### vm.$emit( eventName, …args] )

- **参数**：

  - `{string} eventName`
  - `[...args]`

  触发当前实例上的事件。附加参数都会传给监听器回调。

- 示例：

  #### 只配合一个事件名使用 `$emit`：

  根组件`App.vue`上

  1. 定义+引入

  ```js
  import welcom from "./components/welcom";
  ```

  2. 注册组件

  ```js
  // 注册组件
      components:{
          welcom
      },
  ```

  3. 使用组件

  ```html
  <template>
      <div>
          <!--父组件订阅welcome事件，触发子组件的emit时，父组件会收到发布通知，执行sayHi方法-->
          <welcom v-on:welcome="sayHi"></welcom>
      </div>
  </template>
  ```
```
  
  

子组件`welcom.vue`:

​```html
<template>
    <!--子组件通过$emit发布事件welcome-->
    <button v-on:click="$emit('welcome')">
      Click me to be welcomed
    </button>
</template>
```

#### 配合额外的参数使用 `$emit`：

`emitPro.vue`子组件：

```vue
<template>
    <button v-on:click="giveAdvice">
      Click me for advice
    </button>
</template>
<script>
export default {
    data(){
        return {
             possibleAdvice: ['Yes', 'No', 'Maybe']
        }
    },
    methods:{
         giveAdvice: function () {
             console.log(this);
            var randomAdviceIndex = Math.floor(Math.random() * this.possibleAdvice.length)
            this.$emit('give-advice', this.possibleAdvice[randomAdviceIndex])
        }
    }
}
</script>
```

`App.vue`根组件：

```vue
/* 模板内容 */
<template>
    <div>
        <welcom v-on:welcome="sayHi"></welcom>
        <emitPro v-on:give-advice="showAdvice"></emitPro>
    </div>
</template>

/* JS内容 */
<script>
//1. 定义+引入 2. 注册组件 3. 使用组件
import welcom from "./components/welcom";
import emitPro from "./components/emitPro";

export default {
    // data是一个函数
    data(){
        return {
            age:10
        }
    },
    // 注册组件
    components:{
        welcom,
        emitPro
    },
    methods:{
        sayHi: function () {
            console.log(this);
            alert('Hi!')
        },
        showAdvice: function (advice) {
            console.log("advice:",advice);
            alert(advice)
        }
    }
    
}
</script>

/* 样式内容 */
<style>
</style>
```

子组件中的`$emit(a,b)`第二个参数b，会传给父组件上的：`showAdvice`函数作为参数传进

`this.$emit('give-advice',this.possibleAdvice[randomAdviceIndex]`

```html
<emitPro v-on:give-advice="showAdvice"></emitPro>
showAdvice: function (advice) {
            console.log("advice:",advice);//这里就是b参数的值
            alert(advice)
        }
```

#### 小案例

> 需求：点击子组件的Button，改变父组件的金额。

- 子组件通过$emit发布changeMoney事件

  ```js
   methods:{
          give(){
              // 子组件通过$emit发布changeMoney事件
              this.$emit("changeMoney",1000)
          }
      }
  ```

- 父组件**订阅**changeMoney事件，如果接收到发布通知，则执行change事件

```html
<SonMoney @changeMoney="change"></SonMoney>
```

子组件`SonMoney`:

```vue
<template>
    <div>
        <button @click="give">给钱</button>
    </div>
</template>

<script>
export default {
    methods:{
        give(){
            // 子组件通过$emit发布changeMoney事件
            this.$emit("changeMoney",1000)
        }
    }
}
</script>
```

父组件`Parent.vue`：

```vue
/* 模板内容 */
<template>
    <div class="parent">I am:{{msg}}
        <div>
            父组件：金额：{{money}}元
            <!-- 父组件订阅changeMoney事件，如果接收到发布通知，则执行change事件 -->
            <SonMoney @changeMoney="change"></SonMoney>
        </div>
    </div>
</template>

/* JS内容 */
<script>
// 引入子组件.
import SonMoney from "./SonMoney"
// 通过props data获取数据
export default {
    // data是一个函数
    data(){
        return {
            msg:"parent",
            money:500
        }
    },
    // 注册组件
    components:{
        SonMoney
    },
    methods:{
        change(value){
            // 这里的value是子组件发布事件时$emit(fn,b)的b参数
            if(this.money>3000){
                alert("不能再给了！")
                return
            }
            this.money += value
            alert("给钱1000")
        }
    }
}
</script>

/* 样式内容 */
<style>
    .parent {
        color: aquamarine;
        border: 5px solid rgb(214, 104, 30);
    }
</style>
```

效果：

![image-20201028174910107](https://i.loli.net/2020/10/28/lOGkHcXsZ9Vf5A8.png)

### 通过`$attrs`从父组件传递属性给子组件

> 使用v-bind，批量的把属性从父组件往下传给子组件。

#### vm.$attrs

> 2.4.0 新增

- **类型**：`{ [key: string]: string }`

- **只读**

- **详细**：

  包含了父作用域中不作为 prop 被识别 (且获取) 的 attribute 绑定 (`class` 和 `style` 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (`class` 和 `style` 除外)，并且可以通过 `v-bind="$attrs"` 传入内部组件——在创建高级别的组件时非常有用。

#### 示例：

`Parent.vue`:

> 父组件中，在引入的子组件上，自己设置的普通属性，或者通过`v-bind`设置的动态属性，在子组件通过`$attrs`可以批量接收到。

- 技巧：如果需要传递数字类型的值，只需要在属性前加`:`或者`v-bind`即可。

```vue
/* 模板内容 */
<template>
    <div class="parent">
        <div>$attrs使用：</div>
        <attrs name="lili" :age="18"></attrs>
    </div>
</template>

/* JS内容 */
<script>
// 引入子组件.
import attrs from "./attrs";
// 通过props data获取数据
export default {
    // 注册组件
    components:{
        attrs
    }
}
</script>
```

`attrs.vue`:

```vue
<template>
    <div>
        <span>{{$attrs}}</span>
    </div>
</template>
```

效果：

![image-20201029152226429](https://i.loli.net/2020/10/29/lGTaMwfFnzeJCjd.png)

#### 通过`v-bind`继续往下传递属性

> 在`attrs.vue`中，继续引入子组件，在标签上通过`v-bind`传递属性。当然，你还可以添加自己定义的属性，在子组件中也能全部获取到

`attrs.vue`:

```vue
<template>
    <div>
        <span>{{$attrs}}</span>
        <attrsGrand v-bind="$attrs" other="someAttr"></attrsGrand>
    </div>
</template>

<script>
import attrsGrand from  "./attrsGrand";
export default {
    components:{
        attrsGrand
    }
}
</script>
```

`attrsGrand.vue`:

```vue
<template>
    <div>
        <div>
            attrsGrand:
        </div>
        {{$attrs}}
    </div>
</template>
```

效果：

![image-20201029160319916](https://i.loli.net/2020/10/29/aKAN43DSueEL8sn.png)

如果你想获取单个属性，则需要使用`props`获取。

```vue
<template>
    <div>
        <div>
            attrsGrand:
        </div>
        {{$attrs}}
        <div>
            props:
            <span>{{other}}</span>
        </div>
    </div>
</template>
<script>
export default {
    props:["other"]
}
</script>
```

![image-20201029163234774](https://i.loli.net/2020/10/29/cISDjdiq6vKJk5r.png)

### 通过`$listeners`从父组件传递方法给子组件

> `$listeners`通过`v-on` 批量把方法从父组件往下传给子组件。

### vm.$listeners

> 2.4.0 新增

- **类型**：`{ [key: string]: Function | Array<Function> }`

- **只读**

- **详细**：

  包含了父作用域中的 (不含 `.native` 修饰器的) `v-on` 事件监听器。它可以通过 `v-on="$listeners"` 传入内部组件——在创建更高层次的组件时非常有用。

比如在父组件`Parent.vue`中，给子组件传递了几个方法：

```vue
<attrs  @click="fn" @mousedown="play"></attrs>
```

在`attrs.vue`组件中，就可以使用`$listeners`接收到所有传过来的方法。

- 可以定义一个方法在控制台输出`this.$listeners`，查看所有的方法。

- 可以通过`v-on`继续传递给attrs的子组件：

  ```html
  <attrsGrand  v-on="$listeners"></attrsGrand>
  ```

- 也可以直接运行其中的方法：

  ```vue
  <span>{{$listeners.click()}}</span>
  ```

  ```html
  <button @click="$listeners.click">点击运行$listeners.click</button>
  ```

  

### 通过`provide`&`inject`传递全局信息

> 在父组件中设置`provide`选项，里面的属性相当于全局变量，在子组件，子孙组件中都能直接使用，不用一层层传递，缺点就是难以追溯属性设置的源头。

在`Parent.vue`中设置`provide`选项：

```vue
<script>
export default {
    // 设置provide选项(是一个函数)，传递数据
    provide(){
        return {
            global:"hello world!"
        }
    },
    // 注册组件
    components:{
        attrs
    }
}
</script>
```

在子孙组件中通过`reject`注入后，直接使用：

​	孙组件`attrsGrand.vue`:

```vue
<template>
    <div class="grand">
      <p>provide传递的信息：</p>
      <div>{{global}}</div>
    </div>
</template>
<script>
export default {
    inject:["global"] //填属性名
}
</script>
```

![image-20201029173633159](https://i.loli.net/2020/10/29/Snt5IAGDrLvqbo1.png)

###  通过`$parent`&`children`&`ref $refs`使用父\子组件上的方法\属性

#### `vm.$parent`:

- **类型**：`Vue instance`

- **只读**

- **详细**：

  获取父实例(父组件)，如果当前实例有的话。

> 需求，在子组件上执行父组件中的fn方法。

`Son.vue`子组件：

```vue
<template>
    <div class="child">
        <div>
            <button @click="doFn">执行父组件上的fn方法</button>
        </div>
    </div>
</template> 
<script>
export default {
  methods:{
          doFn(){
              console.log(this.$parent)
              // 通过$parent找到父组件，然后执行fn方法
              this.$parent.fn();
          }
      },
 </script>
```

#### `vm.$children`

- **类型**：`Array<Vue instance>`

- **只读**

- **详细**：

  当前实例的直接子组件。**需要注意 `$children` 并不保证顺序，也不是响应式的。**如果你发现自己正在尝试使用 `$children` 来进行数据绑定，考虑使用一个数组配合 `v-for` 来生成子组件，并且使用 Array 作为真正的来源。

- 注意：可能会有多个子组件，所以选择时需要使用数组下标。

> 需求：在子组件上，运行孙组件的方法。

`Son.vue`:

```vue
template>
    <div class="child">
        <div>
            <button @click="doFn">执行父组件上的fn方法</button>
            <p>````````````````</p>
            <button @click="doGrandFn">执行子组件上的fn方法</button>
        </div>
        <GrandSon></GrandSon>
    </div>
</template> 
<script>
import GrandSon from  "./GrandSon";
export default {
methods:{
        doFn(){
            console.log(this.$parent)
            // 通过$parent找到父组件，然后执行fn方法
            this.$parent.fn();
        },
        doGrandFn(){
            console.log(this.$children);
            this.$children[0].fn();//注意子组件选择索引
        }
    },
 </script>
```

`GrandSon.vue`:

```vue
<script>
export default {
    methods:{
        fn(){
            alert("I am grandSon!")
        }
    }
}
</script>
```

#### `ref $refs`

##### `ref`

> ​	可以获取DOM元素或者组件。

- **预期**：`string`

  `ref` 被用来给元素或子组件注册引用信息。引用信息将会注册在父组件的 `$refs` 对象上。

  - 如果在**普通的 DOM 元素**上使用，引用指向的就是 DOM 元素；
  - 如果用在**子组件**上，引用就指向组件实例：

  ```html
  <!-- `vm.$refs.p` will be the DOM node -->
  <p ref="p">hello</p>
  
  <!-- `vm.$refs.child` will be the child component instance -->
  <child-component ref="child"></child-component>
  ```

##### `vm.$refs`

- **类型**：`Object`

- **只读**

- **详细**：

  一个对象，持有注册过 [`ref` attribute](https://cn.vuejs.org/v2/api/#ref) 的所有 DOM 元素和组件实例。

- **参考**：

  - [子组件 ref](https://cn.vuejs.org/v2/guide/components-edge-cases.html#访问子组件实例或子元素)
  - [特殊 attribute - ref](https://cn.vuejs.org/v2/api/#ref)

##### 例子：

`Son.vue`中执行孙组件上的方法

```vue
<template>
    <div class="child">
        <GrandSon ref="grandSon"></GrandSon>
        <span ref="node">普通DOM元素</span>
        <button @click="refFn">显示/执行ref</button>
    </div>
</template>
<script>
import GrandSon from  "./GrandSon";
export default {
 methods:{
        doFn(){
            console.log(this.$parent)
            // 通过$parent找到父组件，然后执行fn方法
            this.$parent.fn();
        },
        doGrandFn(){
            console.log(this.$children);
            this.$children[0].fn();
        },
        refFn(){
            console.log(this.$refs.grandSon);
            console.log(this.$refs.node);
            this.$refs.grandSon.fn()//
        }
    }
  </script>
```

- 在普通DOM元素上使用ref进行标记，使用`$refs`时则指向这个DOM元素
- 在组件上使用时则指向这个组件

![image-20201029183352294](https://i.loli.net/2020/10/29/LsWEBt1CYh6N4oj.png)

### 兄弟之间传递数据-->`$parent $on $emit`

#### [vm.$on( event, callback )](https://cn.vuejs.org/v2/api/#vm-on)

- **参数**：

  - `{string | Array<string>} event` (数组只在 2.2.0+ 中支持)
  - `{Function} callback`

- **用法**：

  监听当前实例上的自定义事件。事件可以由 `vm.$emit` 触发。回调函数会接收所有传入事件触发函数的额外参数。

- **示例**：

  ```js
  vm.$on('test', function (msg) {
    console.log(msg)
  })
  vm.$emit('test', 'hi')
  // => "hi"
  ```

> 子组件中使用`$parent`当做中间桥梁，在父组件上发布&订阅事件，实现兄弟组件通信。

`Parent.vue`引用并使用两个子组件，把这两个子组件当做兄弟组件。

```js
import Son from "./Son";
import Son2 from "./Son2";
```

`Son.vue`生命周期函数中，监听一个事件：

```js
 mounted(){//生命周期函数
        this.$parent.$on("test",function(data){
            //data是从Son2.vue中通过$emit()的第二个参数传过来的，实现兄弟间传递数据
            console.log(data);
        })
    },
```

`Son2.vue`触发这个事件：

```js
mounted(){//生命周期函数
        this.$parent.$emit("test",1000)//把1000的参数传了过去
    }
```

也就是说，在两个子组件上，通过$parent操作父组件，在父组件上$on监听和$emit触发相应的事件，可以实现把数据传递到兄弟组件中。



### `v-bind .sync`语法糖更新父组件数据

- `.sync` (2.3.0+) 语法糖，会扩展成一个更新父组件绑定值的 `v-on` 侦听器。

在父组件`Parent.vue`中使用：

```html
<SonMoney :myMoney.sync="money"></SonMoney>
```

像v-bind一样，会把`myMoney`这个属性名传到子组件，子组件用props选项可以接收到对应的属性值。

#### 配套使用`uodate:xxx`

子组件可以使用配套的`$emit("update:myMoney",1000)`把父组件上的money值更新。

- `update:`后面的名字写父组件上`v-bind`的名字。

`SonMoney.vue`:

```vue
<template>
    <div>
        <div>:myMoney.sync:传过来的值：{{myMoney}}</div>
        <button @click="give">给钱</button>
        <slot></slot>
    </div>
</template>

<script>
export default {
    props:["myMoney"],
    methods:{
        give(){
            // 子组件通过$emit发布changeMoney事件
            // this.$emit("changeMoney",1000)
            if(this.$parent.money==1000){
                alert("已经是1000了")
                return
            }
            this.$emit("update:myMoney",1000)
        }
    }
}
</script>
```

![image-20201030164554658](https://i.loli.net/2020/10/30/z6FdaBNDL5qmgOv.png)

### 使用`v-model`传递数据

`Parent.vue`中在子组件上使用v-model:

```html
<SonMoney v-model="money"></SonMoney>
```

而`v-model`是一个语法糖，实际上是`v-bind="value"`与`v-on="input"`一起使用实现的，分开写可以自己控制更多的操作，如：

```html
 <SonMoney :value="money" @input="(data)=>money+=data"></SonMoney>
```

`SonMoney.vue`中：

在`props`中监听的是`value`属性，发布事件的名字是`input`：

```vue
<script>
export default {
    // props:["myMoney"],
    props:{
        value:{
            type:Number //类型检测
        }
    },
    methods:{
        give(){
            this.$emit("input",1000)
        }
    }
}
</script>
```



### 大项目使用vuex兄弟间传递

### 如何修改父组件传递过来的值？

> 子组件中通过`props`接收的值，不能直接修改。

1. 在子组件中，通过`computed`进行属性计算：

   ```js
   <div>修改后的值：{{bigMoney}}</div>  
   computed:{
           bigMoney(){
               console.log("this.value",this.value);
               return this.value +888
           }
       },
   ```

2. 如果传过来的是一个对象，则可以对对象里的成员进行修改，因为指向的是同一个堆内存。
3. 通知父组件，在父组件中进行修改

### 总结

组件间通信方法：

- 父传子：
  - 标签属性+props  传递单个属性
  - `$attrs` 批量传递属性
  - `$listeners` 批量传递方法
  - `provide`&`inject` 设置全局属性，可传递到任何子孙组件
    - 优点：方便，不用一层层传递
    - 缺点：属性多的时候难以追溯是哪个源头
  - `vm.$children` & `vm.$parent`  获取父组件&子组件
  - `vm.$refs` & `ref` 获取组件实例
  - `v-bind .sync`语法糖更新父组件数据 (父子互传)
  - 使用`v-model`传递数据 (父子互传)
- 子传父
  - `$emit()`  通过事件方式传递
- 兄弟传递
  - 通过父组件作为桥梁，使用`$on $emit`事件传递。
  - vuex

