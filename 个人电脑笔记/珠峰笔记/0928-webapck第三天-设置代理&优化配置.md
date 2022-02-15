[TOC]



## 0928-webapck第三天-设置代理&优化配置

### ESLint

> ESLint 是在 ECMAScript/JavaScript 代码中识别和报告模式匹配的工具，它的目标是保证代码的一致性和避免错误。在许多方面，它和 JSLint、JSHint 相似，除了少数的例外：
>
> - ESLint 使用 [Espree](https://github.com/eslint/espree) 解析 JavaScript。
> - ESLint 使用 AST 去分析代码中的模式
> - ESLint 是完全插件化的。每一个规则都是一个插件并且你可以在运行时添加更多的规则。
>
> 中文地址：https://eslint.bootcss.com/docs/user-guide/getting-started
>
> 英文官网：https://eslint.org/docs/user-guide/getting-started

安装ESLint：

```
npm install eslint --save-dev 
# or
yarn add eslint --dev
```

安装ESLint-loader:

```
npm install eslint-loader -D
```

初始化：

```
$ npx eslint --init
# or
$ yarn run eslint --init
```

![image-20200928205803335](https://i.loli.net/2020/09/28/MlTiU3pyK7EvnOR.png)

配置eslint规则，在rules里面添加，可以去官网查看都有哪些规则，按需添加，规则的ID按以下值设置：

> - `"off"` 或 `0` - 关闭规则
> - `"warn"` 或 `1` - 开启规则，使用警告级别的错误：`warn` (不会导致程序退出)
> - `"error"` 或 `2` - 开启规则，使用错误级别的错误：`error` (当被触发的时候，程序会退出)

```js
module.exports = {
    "env": {
        "browser": true,
        "es2021": true
    },
    "extends": "eslint:recommended",
    "parserOptions": {
        "ecmaVersion": 12,
        "sourceType": "module"
    },
    "rules": {
        "no-console":2 
    }
};
```

在weboack.base.js配置loader，注意eslint需要最优先加载

```js
 {
   test: /\.js$/,
   use: "eslint-loader",
   enforce: "pre" //最优先加载
 }
```

在index.js写以下代码：

```js
let a = b;
let c = 1 + 2;
console.log("ok")
```

npm run dev后会显示报错：

![image-20200928212842293](https://i.loli.net/2020/09/28/VrFAqJTpUDCKyv1.png)

### 删除无用的CSS

需要安装的包：

- purgecss-webpack-plugin  
- glob 设置查找文件的路径和匹配的文件(要删除的文件路径在哪)
- glob-all （推荐使用）可以设置匹配多条文件路径

安装：

```
npm install purgecss-webpack-plugin glob-all -D
```

本次的案例还需要用到react相关的包，也需要安装一下

```
npm install react react-dom @babel/preset-react -D
```

在index.js里：

```js
import React from "react";
import ReactDOM from "react-dom";
ReactDOM.render(<h1 className="box">hello world</h1>, document.getElelmentById("root"));
```

在`.babelrc`使用预设：

```json
{
    "presets": [
        "@babel/preset-react"
    ]
}
```

在`webpack.prod.js`:

```js
//引入 
const PurgecssPlugin = require("purgecss-webpack-plugin");
const glob = require("glob");
//插件设置如下
plugins: [
        new PurgecssPlugin({
            paths: glob.sync(`${path.resolve(__dirname, "src")}/**/*`, { nodir: true }),  //查找src下任意文件，不包含文件夹
        })
    ]
```

匹配多个文件路径：

> 如果你要匹配html中的类名，就要设置匹配在根目录下的html文件；
>
> `${path.resolve(__dirname)}/*.html`或者`path.join(__dirname, './*.html')`

```js
new PurgecssPlugin({
            paths: glob.sync([
                path.join(__dirname, './*.html'),//匹配在根目录下的html文件
                path.join(__dirname, './src/*.js'),
                path.join(__dirname, './src/*.ts'),
            ], {
                    nodir: true
                })
        })
```

在`webpakc.base.js`:

```js
//引入
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
//loader使用
 rules: [
    {
      test: /\.css$/,
      use: [
          {
            loader: isDev ? "style-loader" : MiniCssExtractPlugin.loader
          },
          {
            loader: "css-loader"
          }

        ]
    },
    {
      test: /\.js$/,
      use: "babel-loader",
      exclude: /node_modules/
    }
 ]
```

#### 注意事项

		- 如果你使用的css类名是单个字母，则purgecss-webpack-plugin不会帮你清除掉
		- 如果你src下有文件夹，同时你又使用了单个字母作为类名，把所有文件夹删除掉即可正常删除这些类名

可能这是一些还没修复的Bug，使用时需要注意。

### CND加载文件

> 有时候我们会引入CDN外部的地址，在index.js中我们想告诉别人"$"是从哪里来的，需要写`import $ from "jquery"`，但是这样打包时会从node_modules下寻找jq，我们不希望wabpack把jq打包进去，这时候需要设置`externals`参数。
>
> 微软jquery地址：https://ajax.aspnetcdn.com/ajax/jquery/jquery-3.4.1.min.js

`webpack.base.js`中，添加`externals`参数：

externals排除打包node_modules下的第三方包。

格式："包的名字" : "这个包的别名"

```
externals: {
            "jquery": "$"   //设置不要打包从外部引入的jquery
        },
```

在index.js中：

```js
import $ from "jquery"  //为了说明$来自哪里，并不希望打包时还打包
```

这样就不会把jquery也打包进去了。

没有设置externals前的打包大小：

![image-20200929122327644](https://i.loli.net/2020/09/29/RedgZ6aLs23lU5j.png)

配置好后的大小：

![image-20200929122406413](https://i.loli.net/2020/09/29/NA6iv7yJ9EO8UYM.png)

### Tree-shaking & scope-hosting

> Tree-shaking ：顾名思义，把无用的东西摇晃掉。
>
> 这是唯一一个需要在`package.json`中设置的参数。--> `"sideEffects": false,`
>
> 这个参数只有在生成环境下打包才生效，开发环境不生效。

#### Tree-shaking使用

我们建两个文件，一个使用，一个不使用，作对比：

sum.js:

```js
const add = (n, m) => {
    return n + m
}

const minus = (n, m) => {
    return n - m
}

export { add, minus }
```

count.js:

```js
export const count = () => {
    console.log("hello world")
}
console.log(count());//这里直接使用一下
```

在index.js中，两个文件都引入进来，但是其中一个没有使用

```js
import "./style.css"
import count from "./count";
import { add } from "./sum";
console.log(add(2 , 8))
```

然后在`package.json`中设置`"sideEffects": false`。

前后对比，没有设置前，会输出hello world

![image-20200929124656965](https://i.loli.net/2020/09/29/VfYd8jQT7MztwP2.png)

设置参数后：

![image-20200929124726155](https://i.loli.net/2020/09/29/CxiIVglZNomaUJy.png)

#### 副作用：会把css也摇晃掉了

> 直接设置`sideEffects:false`的副作用是会把在index.js里引入的.css文件也会删掉，我们需要设置排除.css的文件。

`package.json`里：

```js
"sideEffects": [
    "**/*.css" //排除任意模块下的后缀名为,css的文件
  ],
```

设置好后，css文件就能正常打包了。

#### 设置开发环境下显示使用了哪个方法/模块

正常情况下，运行npm run dev 在bundle.js里只能看到供了哪个模块，但是没有显示我使用了哪个模块，我们可以设置成显示使用了哪个模块。

`webpack.base.js`添加以下参数:

```js
// 设置显示使用了哪个模块
optimization: {
 	usedExports: true
},
```

没有设置前：

![image-20200929140729407](https://i.loli.net/2020/09/29/JOmq1Ew3ayUDRGd.png)

设置后：

![image-20200929141348943](https://i.loli.net/2020/09/29/c48FDlxOm2RhnwW.png)

#### scope-hosting使用

> scope-hosting的作用：提升作用域，减少代码体积，节约内存，代码量明显减少，
>
> 多个函数内存的占用也减少。

比如在count.js下：

```js
let a = 1;
let b = 2;
let c = 3;
let d = a + b + c;
export default d; 
```

在index.js下调用：

```js
import count from "./count";
console.log(count)
```

使用scope-hosting优化后，会直接导出d的结果。

在webpaack4以上的版本自带scope-hosting，在生产环境下打包会自动帮你优化。

打包结果显示，直接输出d的结果6

![image-20200929143715904](https://i.loli.net/2020/09/29/WTI819HpgKL45eh.png)

### 配置devServer代理

> 场景：一个项目中可能会有多个模块，每个模块下的请求api地址都会有不同的标识，比如a模块下的地址是"/api/user"，b模块下的地址是"/api2/user"，但是后端提供的地址是"/user"，这时候我们就需要配置下代理。

index.js下有一个请求：

```js
// 发送请求
let xhr = new XMLHttpRequest();
xhr.open("get", "api/user", true);
xhr.onreadystatechange = function () {
    console.log(xhr.response);
}
xhr.send();
```

后端接口：

```js
let express = require("express");//node自带的
let app = express();
app.get("/user", function (req, res) {
    // req是请求时携带的信息，res是响应时携带的信息
    console.log(req.headers);
    res.json({ name: "lili" });//以JSON格式返回一个数据
})
app.listen(9090, function () {
    console.log("9090端口已启动")
})
```

`webpack.dev.js`下的devServer配置：

```js
 devServer: {
        port: 9999, //端口号
        open: false, //自动打开窗口
        compress: true, //开启gizp压缩
        hot: true, //css代码热更新，只支持style标签的css代码
        contentBase: path.resolve(__dirname, "static"), //以dist为目录，访问该目录的静态资源
        // 设置代理
        proxy: {
            "/api": {
                target: "http://localhost:9090",//请求服务器的地址
                secure: false, //true表示https , false表示http
                changeOrigin: true, //把请求头中的host值改成服务器的地址
                pathRewrite: { "/api": "" }//重写请求的接口地址
            }
        }
    }
```

 **changeOrigin: true**

 设置后可以把请求头中的host值改成服务器的地址，因为有的网站后端可能会有判断，如果你的host不是服务器的地址，则不允许跨域等的拦截。

设置前：

![image-20200929162542023](https://i.loli.net/2020/09/29/odmUpfgZv3R6rHM.png)

设置后：

![image-20200929162604939](https://i.loli.net/2020/09/29/ln5mc9ogTzrLOpF.png)

 **pathRewrite: { "/api": "" }**

代理proxy中写了target后，因为前端请求的地址是"/api/user"，与后端接口不同，所以还需要重写请求的接口地址。

### devServer中模拟返回数据

> 除了我们自己写一个后端返回数据以外，在前端devServer中也可以模拟返回数据。

`webpack.dev.js`中配置：

```js
 devServer: {
        port: 9999, //端口号
        open: false, //自动打开窗口
        compress: true, //开启gizp压缩
        hot: true, //css代码热更新，只支持style标签的css代码
        contentBase: path.resolve(__dirname, "static"), //以dist为目录，访问该目录的静态资源
        before: function (app, server) {
            //app就相当于node中开启express服务的app server指的就是当前devServer
            app.get("/api/user", function (req, res) {
                res.json({ "name": "lili" })
            })
        }
 }
```

就是相当于在index.js中向"/api/user"发送请求时，在devServer中会捕获这个请求，并返回数据，是同域的方式。

### DllPlugin && DllReferencePlugin

> DllPlugin 设置动态链接库
>
>  DllReferencePlugin  引用动态链接库
>
> 这两个库都是webpack自带的。
>
> 作用：把第三方库或者框架 单独打包成动态链接库  （保存下）每次再打包整个项目时 ，不需要再打包第三方库或者框架，只需要引用下就行。

#### 打包所有的库

新建一个`webpack.dll.js`:

```js
const path = require("path");
module.exports = {
    mode: "development",
    entry: ["react", "react-dom"],
    output: {
        library: "react",//用来接收打包后自执行函数返回值的名称
        //libraryTarget: "commonjs2",//用哪种方式来接收自执行函数返回值：有commonjs commonjs2 this var umd ，默认通过var接收
        path: path.resolve(__dirname, "dll"),//输出的路径
        filename: "react.dll.js"  //输出的名字
    }
}
```

在`package.json`中设置script命令：

```
"dll":"webpack --config ./webpack.dll.js"
```

运行命令后，就会在dll目录下生成一个react.dll.js的文件，里面会用`var react `来接收自执行函数返回的值。

![image-20200929173235521](https://i.loli.net/2020/09/29/zY92KO31gEMlrmI.png)

接下来就要建一个缓存列表，用来存储react.dll.js中包含的各种复杂的库，相当于建立一个索引，在我们需要调用时，直接通过调取这个缓存列表文件的某个库地址，就能成功调用，不用每次都打包。

缓存列表文件和react.dll.js文件的关系图：

![image-20200929173701787](https://i.loli.net/2020/09/29/BJy9s7dTlNGSt2j.png)

#### 生成dll缓存列表文件

`webpack.dll.js`中使用DllPlugin插件：

```js
const path = require("path");
const DllPlugin = require("webpack").DllPlugin;//引入动态链接库
module.exports = {
    mode: "development",
    entry: ["react", "react-dom"],
    output: {
        library: "react",//用来接收打包后自执行函数返回值的名称
        //libraryTarget: "commonjs2",//用哪种方式来接收自执行函数返回值：有commonjs commonjs2 this var umd ，默认通过var接收
        path: path.resolve(__dirname, "dll"),//输出的路径
        filename: "react.dll.js"  //输出的名字
    },
    plugins: [
        // 调用DllPlugin插件
        new DllPlugin({
            name: "react",
            path: path.resolve(__dirname, "dll/mainfest.json")
        })
    ]
}
```

#### 缓存列表与库建立关系

首先，我们需要把生成的`react.dll.js`引入到html页面中，

这里需要安装一个插件用来引入文件：

add-asset-html-webpack-plugin  -->用来把js文件添加到html当中

```
npm install add-asset-html-webpack-plugin -D
```

在`webpack.base.js`中使用这个插件

```js
// 用来引入文件到html中的插件
const AddAssetHtmlPlugin = require("add-asset-html-webpack-plugin");

//添加插件
new AddAssetHtmlPlugin({
  filepath: path.resolve(__dirname, "dll/react.dll.js")
}),
```

打包后，会自动把文件插入到html中，并且在dist中拷贝一份js

![image-20200929180049894](https://i.loli.net/2020/09/29/8Cswh2auZckjme6.png)

然后使用`DllReferencePlugin`插件把我们的JS库文件和缓存列表文件建立关联。

`webpack.base.js`：

```js
// 建立缓存关系的插件
const DllReferencePlugin = require("webpack").DllReferencePlugin
//使用一下插件
plugins: [
            //建立缓存关系的插件
            new DllReferencePlugin({
                // 告诉插件你的缓存列表文件是谁
                manifest: path.resolve(__dirname, "dll/mainfest.json")
            }),
        ].filter(Boolean)
```



#### 前后对比

使用插件前bundle.js为992kb：

![image-20200929181326954](https://i.loli.net/2020/09/29/E6sGobLagFtCD89.png)

使用插件优化后，bundle仅为21.8kb，打包后的文件体积大幅减少，代码量也大幅减少：

![image-20200929181419718](https://i.loli.net/2020/09/29/aWfHb41wlkzZspN.png)

### 动态加载文件

> 点击按钮后才加载函数所在的文件，不用在加载页面的时候就加载。

`index.js`:

```js
// 动态加载文件
let btn = document.createElement("button");
btn.addEventListener("click", () => {
    import("./sum").then(data => {
        console.log(data.add(10, 20));
    })
})
btn.innerHTML = "点我加载";
document.body.appendChild(btn);
```

如果不设置，默认的名字是`0.xxx.js`，name从0开始，xxx是你输出到dist目录的js文件

#### 设置输出名称

#### 普通设置名称

在`webpack.base.js`:

```js
 let base = {
        entry: "./src/index.js", //webpack的入口文件
        output: {
            // 动态加载文件的名称设置,name默认从0开始
            chunkFilename: "[name].mini.js",
            path: path.resolve(__dirname, "dist"),
            filename: "bundle.js" //打包后的输出文件名
        }，
        ...
       }
```

在点击按钮后，文件开始加载

![image-20201005183754244](https://i.loli.net/2020/10/05/QhanP1Fflme7bS2.png)

#### 进阶设置名称

+ `webpackChunkName`  动态加载文件的名称

+ `webpackPrefetch` 预加载   利用浏览器空闲的时间，把动态模块加载完成并引入进来

+ `webpackPreload` 预拉取  跟主模块的代码同时加载，并行的下载文件

#### 使用方法：

> 在使用的地方`import()`里设置名称

`webpackChunkName`：

```js
// 动态加载文件
let btn = document.createElement("button");
btn.addEventListener("click", () => {
  //webpackChunkName:'name' 设置name名称
    import(/*webpackChunkName:'sum'*/"./sum").then(data => {
        console.log(data.add(10, 20));
    })
})
btn.innerHTML = "点我加载";
document.body.appendChild(btn);
```

![image-20201005184705151](https://i.loli.net/2020/10/05/P5Gd9jQHvYniCKp.png)

`webpackPrefetch`：在浏览器空闲的时候就加载，还没点击就加载（前面的资源以及加载后）

```js
btn.addEventListener("click", () => {
    import(/*webpackPrefetch:true*/"./sum").then(data => {
        console.log(data.add(10, 20));
    })
})
```

![image-20201005185055142](https://i.loli.net/2020/10/05/NK2Hm95xp3OvgtB.png)

`webpackPreload`：和主页面的文件一起加载，已经下载了还没引入，在你点击时才引入

```js
btn.addEventListener("click", () => {
    import(/*webpackPreload:true*/"./sum").then(data => {
        console.log(data.add(10, 20));
    })
})
```

![image-20201005185425509](https://i.loli.net/2020/10/05/yuLCMoRUdmvzjWE.png)

### devServer的热更新

> 在开发环境下，使用devServer预览时，可以设置参数`hot:true`支持热更新，但是只能识别style标签里的css样式变化，不能识别link标签引入的css。这里已经通过`css-loader style-loader`把样式添加到style标签上。

#### CSS热更新

```js
const path = require("path")
module.exports = {
    mode: "development", //添加环境 development production none 三者其一
    devServer: {
        port: 9999, //端口号
        open: false, //自动打开窗口
        compress: true, //开启gizp压缩
        hot: true, //css代码热更新，只支持style标签的css代码
        contentBase: path.resolve(__dirname, "static"), //以dist为目录，访问该目录的静态资源

    }
}
```

在css文件中修改样式，可以不用刷新页面就能立刻更新。

![image-20201005191530663](https://i.loli.net/2020/10/05/wxNtSHR31T9bI47.png)

#### JS热更新

> 通过module.hot.accept强制支持热更新

`index.js`:

```js
import "./style.css"
import { add } from './sum';
let p = document.createElement("p");
p.innerHTML = add(10, 22);
document.body.appendChild(p)
// 让JS文件支持热更新
if (module.hot) {
    module.hot.accept("./sum", () => {
        p.innerHTML = add(10, 22)
    })
}
```

![image-20201005205005644](https://i.loli.net/2020/10/05/U2VfR5eKbHX8YWc.png)

### sourceMap(代码排查工具)

> 可以准确的定位到错误是哪一行，页面的JS代码跟自己的源文件一样

- 开发环境 `cheap-module-eval-source-map` 
- 生产环境  `cheap-module-source-map`



添加后：

![image-20201005220421540](https://i.loli.net/2020/10/05/v1LCuw7yxKfHdMi.png)

![image-20201005221141444](https://i.loli.net/2020/10/05/xugnSmhUFTq59Pl.png)

![image-20201005221124869](https://i.loli.net/2020/10/05/AwcZmF8jnvLTrH3.png)

### 打包文件分析工具 webpack-bundle-analyzer

> 在生产环境下使用，用来分析我们的代码，提高代码质量

安装：

npm install webpack-bundle-analyzer -D

### 抽离模块插件-splitChunks

> 配合 webpack-bundle-analyzer 使用，可以查看打包后的模块大小
>
> 生产环境下使用，在编译时抽离第三方模块或公共模块，不要和dllPlugin同时使用
>
> https://webpack.js.org/plugins/split-chunks-plugin/#optimizationsplitchunks

`webpack.prod.js`: 设置splitChunks

```js
module.exports = {
    mode: "production", //添加环境 development production none 三者其一
    devtool: "cheap-module-source-map",
    optimization: {
        splitChunks: {
            chunks: 'all',  //all 同步+异步  async 异步模块
            minSize: 20000, //分割文件的最小大小
            maxSize: 0,
            minChunks: 1,//引用的次数
            maxAsyncRequests: 30,//最大的异步请求
            maxInitialRequests: 30,//最大的初始化请求数
            automaticNameDelimiter: '~', //抽离的命名的分隔符
            name: true,
            cacheGroups: {
                // 自定义名字
                reactVendor: {
                    test: /[\\/]node_modules[\\/](react)|(react-dom)/,
                    // 优先级
                    priority: -1
                },
                // 自定义名字：jqueryVendor
                jqueryVendor: {
                    minChunks: 1,
                    priority: -2
                }
            }
        }
    }
   }
```

`webpack.base.js`:设置多入口测试多个js模块

注意：使用本地jquery时不要设置externals参数

```js
// 多入口
        entry: {
            index: './src/index.js',
            other: './src/other.js'
        },
        output: {
            // 动态加载文件的名称设置,name默认从0开始
            chunkFilename: "[name].mini.js",
            path: path.resolve(__dirname, "dist"),
            filename: "[name].js" //打包后的输出文件名
        },
```

`index.js`和`other.js`下引入一下jquery

```js
import $ from "jquery"
console.log($)
```

npm run build运行

效果：

![image-20201005225620068](https://i.loli.net/2020/10/05/OoW3IfytQ24zNMj.png)

引入react

```js
import React from "react";
import ReactDOM from "react-dom";
ReactDOM.render(<h1 className="box">hello world</h1>, document.getElementById("root"));
```

![image-20201005230101477](https://i.loli.net/2020/10/05/AXKUu4QRW6BgcCH.png)

### 费时分析 speed-measure-webpack-plugin

> https://www.npmjs.com/package/speed-measure-webpack-plugin

安装：

```
npm install speed-measure-webpack-plugin -D
```

`webpack.base.js`引入

```js
// 费时分析工具
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");
const smp = new SpeedMeasurePlugin();
```

使用的时候，用`smp.wrap()`包裹一下：

```js
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");
 
const smp = new SpeedMeasurePlugin();
 
const webpackConfig = smp.wrap({
  plugins: [
    new MyPlugin(),
    new MyOtherPlugin()
  ]
});
```

也可以用在不同环境下使用的插件分析

```js
 if (env.development) {
        // base和开发配置dev进行合并
        return smp.wrap(merge(base, dev))
    } else {
        // base和生产配置prod进行合并
        return smp.wrap(merge(base, prod))
    }
```

![image-20201005231205549](https://i.loli.net/2020/10/05/CTrJVzN54pukoAE.png)

### resolve路径优化

> 可以提升构建时查找模块的速度

- extensions  设置后引用时可以省略文件后缀
- alias  设置别名 在项目中可缩减引用路径

`webpack.base.js`:

```js
 resolve: {
            extensions: ['.js', '.jsx', ,'.vue','.json', '.css'],
            alias: {//设置别名 在项目中可缩减引用路径
                "@": path.resolve(__dirname, "src")
            }
        },
```

#### resolve.alias

这样配置后在使用的时候就可以直接

```js
import http from '@/utils/http'

代替

import http from 'src/utils/http'
```

#### resolve.extensions

通过这样的配置，我们在组件中过着路由中应用组件时，就可以更为方便的应用，比如：

```js
  import Hello from '@components/Hello';
```

　　即Hello.vue这个组件我们不需要添加.vue后缀就可以引用到了，如果不用extensions， 我们就必须要用@components/Hello.vue来引入这个文件。 

参考资料：https://www.cnblogs.com/chenguiya/p/10661529.html

### noParse 设置不解析的模块

`webpack.base.js`

```js
let base = {
  	noParse:/jquery/, //设置哪些模块不要解析
}
```

### IgnorePlugin 忽略特定的模块

防止在 `import` 或 `require` 调用时，生成以下正则表达式匹配的模块：

- `requestRegExp` 匹配(test)资源请求路径的正则表达式。
- `contextRegExp` （可选）匹配(test)资源上下文（目录）的正则表达式。
- 第一个regexp必须与该`'./locale/'`字符串匹配。然后，第二个`contextRegExp`参数用于选择从中进行导入的特定目录

```js
new webpack.IgnorePlugin(requestRegExp, [contextRegExp])
```

#### 比如忽略moment这个模块：https://momentjs.com/

```js
忽略 moment 的本地化内容
moment 2.18 会将所有本地化内容和核心功能一起打包（见该 GitHub issue）。你可使用 IgnorePlugin 在打包时忽略本地化内容:

new webpack.IgnorePlugin({
  resourceRegExp: /^\.\/locale$/,
  contextRegExp: /moment$/
});
```

```js
new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/)
```

### happypack  多线程的处理

> https://www.npmjs.com/package/happypack

安装：

```
npm install happypack -D
```

#### 插件里的使用

```js
// @file webpack.config.js
exports.plugins = [
  new HappyPack({
    id: 'jsx',
    threads: 4,
    loaders: [ 'babel-loader' ]
  }),
 
  new HappyPack({
    id: 'styles',
    threads: 2,
    loaders: [ 'style-loader', 'css-loader', 'less-loader' ]
  })
];
```

rules里的使用

```js
exports.module.rules = [
  {
    test: /\.js$/,
    use: 'happypack/loader?id=jsx'
  },
 
  {
    test: /\.less$/,
    use: 'happypack/loader?id=styles'
  },
]
```

