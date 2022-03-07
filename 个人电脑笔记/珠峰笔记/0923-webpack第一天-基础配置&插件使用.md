[TOC]



## webpack第一天-基础配置&插件使用

### webpack的安装

1. 首先创建一个目录；
2. 在目录的终端输入：`npm init -y` 初始化npm；
3. 在本地安装(开发环境安装)webpack和webpack-cli(此工具用于在命令行中运行 webpack)，不要在全局下安装，不然只能使用全局的一个版本。

```cmd
mkdir webpack_day1 && cd webpack_day1
npm init -y
npm install webpack webpack-cli --save-dev(或者-D)
```

### webpack初体验

​	在我们的目录下，创建一个src文件夹，有以下文件：

```
-/
	-src
		-index.js
		-test1.js
```

`test1.js`

```js
module.exports = "hello world!"
```

`index.js`

```js
let word = require("./test1")
```

如果在根目录下的index.html文件想引入这个src下的index.js文件，浏览器是不能输出的，因为Node的语法不适用与浏览器。

![image-20200923234922937](https://i.loli.net/2020/09/23/a62MujTRNyC8lt3.png)

这时候就需要使用webpack帮我们打包成支持浏览器的语法格式。

webpack支持零配置运行，由于我们不是安装在全局，所以我们使用npx命令执行一下。

```
npx webpack
```

![image-20200924003703196](https://i.loli.net/2020/09/24/z28WlmrRbBxjopg.png)

会出现一个新目录：`dist`，里面是打包好的文件。

**注意：**这里有个坑，使用npx命令时，你所建的目录不能太深，最好在桌面新建一个文件夹在里面运行，不然会报找不到webpack.js这个包的错误。

接下来就把根目录下的index.html里的script标签里的src路径换成`./dist/main.js`,然后就能在浏览器输出啦。

![image-20200924004221961](https://i.loli.net/2020/09/24/Ljw1IOfJFb2G8qA.png)

#### 关于`npx`命令

> npx webpack  npx是5.2版本以后npm提供的命令

使用`npx webpack`会找到node_modules下.bin下可执行文件webpack命令，命令内部会调用webpack-cli解析用户参数进行打包 。

### 配置npm的srcipt命令

经过上面的操作，打包后会有一个提示：

![image-20200924152742072](https://i.loli.net/2020/09/24/v21BzlnP3yJLquK.png)

这个提示的意思是告诉我们需要设置一个`mode`去指定开发环境还是生产环境，接下来就来设置一下。

​	打开`package.json`文件，在`script`下设置两个命令：

```json
 "scripts": {
    "dev": "webpack --mode development",
    "build": "webpack --mode production"
  },
```

设置后如何使用呢？在终端下输入以下命令，代表不同环境下运行，

```js
npm run dev //开发环境下运行

npm run build //生产环境下运行
```

`npm run dev`运行这个命令，webpack不会压缩main.js文件；

`npm run build`运行这个命令，webpack会压缩main.js文件(去掉注释和空格)



### webpack的配置文件

在根目录下创建一个`webapck.config.js`文件，在里面写我们需要的配置。

webapck.config.js：

> webapck.config.js文件下遵循的是commonJS语法规范，跟node一样。

#### 入口(entry)

​		**入口起点(entry point)**指示 webpack 应该使用哪个模块，来作为构建其内部 [依赖图(dependency graph)](https://webpack.docschina.org/concepts/dependency-graph/) 的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。

默认值是 `./src/index.js`，但你可以通过在 webapck.config.js中配置 `entry` 属性，来指定一个（或多个）不同的入口起点。

#### 输出(output)

​		**output** 属性告诉 webpack 在哪里输出它所创建的 *bundle*，以及如何命名这些文件。主要输出文件的默认值是 `./dist/main.js`，其他生成文件默认放置在 `./dist` 文件夹中。

你可以通过在配置中指定一个 `output` 字段，来配置这些处理过程。

#### 示例用法

```js
const path = require("path")
module.exports = {
    entry: "./src/index.js", //webpack的入口文件
    output: {
        path: path.resolve(__dirname, "dist"), //需要写绝对路径，用到node的全局变量__dirname(当前文件所在的路径)，与我们指定的目录dist拼接
        filename: "bundle.js" //打包后的输出文件名
    }
}
```

- `__dirname`：是node环境下的全局变量，用来获取当前文件所在的路径。
- `path.resolve()`：是node核心模块path下的一个方法，用来合并路径的。

### 使用自定义配置文件

> 如果我们想自己创建一个配置文件，用不同的命令来输出不同的配置文件该怎么做呢？

#### mode(模式)

​		这里新增加一个知识点，`mode`在webpack配置文件中，可以指定使用的环境进行输出。通过选择 `development`, `production` 或 `none` 之中的一个，来设置 `mode` 参数，你可以启用 webpack 内置在相应环境下的优化。其默认值为 `production`。

首先，在根目录下创建`webpack.dev.js`

```js
const path = require("path")
module.exports = {
    entry: "./src/index.js", //webpack的入口文件
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "bundle2.js" //打包后的输出文件名
    },
    mode: "development"
}
```

然后，在package.json文件script下添加一条命令：

```json
"scripts": {
    "dev": "webpack --mode development",
    "build": "webpack --mode production",
  + "myDev": "webpack --config ./webapck.dev"
  },
```

`webpack --config ./webapck.dev`通过--config参数，让webpack寻找指定的配置文件。

最后，运行`npm run myDev`，就可以输出bundle2.js文件了。

### 开发环境与生产环境运行不同的配置

> 通过上面的操作，我们可以得到启示，我们完全可以自己配两套配置，用于开发环境和生产环境，只需要运行不同的命令就能控制不同环境的输出。

package.json:

```json
"scripts": {
    "dev": "wabpack --config ./wabpack.dev",
    "build": "wabpack --config ./wabpack.prod"
  }
```

根目录下创建：`webpack.dev.js`、`webpack.prod.js`两个文件，里面配置开发环境与生产环境的不同配置，如`mode`的不同。

使用的时候：

```
npm run dev 
npm run build 
```

这就能实现我们需要的功能了。

### 公共配置文件

> 真实项目中，我们的开发环境和生产环境会有公共的配置部分，如果在dev和prod文件都写一遍，这样就不太好，我们需要把公共部分抽离处理，运行`npm run`时通过传递的参数不同，实现按照：公共+开发or生产进行打包。

我们在根目录下新建一个`webpack.base.js`公共配置文件。

我们如何在运行公共配置文件时，区分开发环境还是生产环境呢？这里我们需要在命令里**传递参数**进行判断。

在`package.json`文件下：

```json
"scripts": {
    "dev": "webpack --env.development --config ./webpack.base",
    "build": "webpack --env.production --config ./webpack.base"
  }
```

这里我们需要安装一个配置合并工具，

```
npm install webpack-merge -D  
```

在`webpack.base.js`下写入：

```js
const path = require("path")
const dev = require("./webpack.dev")
const prod = require("./webpack.prod")
const {
    merge
} = require("webpack-merge") //导入merge工具包
module.exports = (env) => {
    console.log(env); //输出环境 { development: true } / { production: true }

    let base = {
        entry: "./src/index.js", //webpack的入口文件
        output: {
            path: path.resolve(__dirname, "dist"),
            filename: "bundle2.js" //打包后的输出文件名
        },
        mode: "development" //添加环境 development production none 三者其一
    }
    if (env.development) {
        // base和开发配置dev进行合并
        return merge(base, dev)
    } else {
        // base和生产配置prod进行合并
        return merge(base, prod)
    }
}
```

这样，就能把公共配置和不同环境配置进行合并处理了。如果两个配置有重合的地方，以传递参数的环境配置优先。

### 清除输出目录插件clean-webpack-plugin

> https://www.npmjs.com/package/clean-webpack-plugin

首先安装插件：

```
npm install clean-webpack-plugin -D  //安装插件
```

在`webpack.config.js`文件下：

```js
const path = require("path")
// 引入插件
const {
    CleanWebpackPlugin
} = require("clean-webpack-plugin")
module.exports = {
    entry: "./src/index.js", //webpack的入口文件
    output: {
        path: path.resolve(__dirname, "dist"), //需要写绝对路径，用到node的全局变量__dirname(当前文件所在的路径)，与我们指定的目录dist拼接
        filename: "bundle.js" //打包后的输出文件名
    },
    plugins: [
        new CleanWebpackPlugin()
    ]
}
```

然后每次运行`npm run dev`后，就会把dist目录下其他文件清空。

#### 清除指定目录下的文件

我们只需在CleanWebpackPlugs添加配置：

第一个参数是需要清除的目录，第二个是不清除的文件名，也可以不写。设置后只清除这个目录里面的文件，其他文件是不清除的。

```
cleanOnceBeforeBuildPatterns: ['**/*', '!static-files*'],//第一个参数是需要清除的目录，第二个是不清除的文件名，也可以不写。
```

在`webpack.config.js`文件下:

```
const path = require("path")
// 引入插件
const {
    CleanWebpackPlugin
} = require("clean-webpack-plugin")
module.exports = {
    entry: "./src/index.js", //webpack的入口文件
    output: {
        path: path.resolve(__dirname, "dist"), //需要写绝对路径，用到node的全局变量__dirname(当前文件所在的路径)，与我们指定的目录dist拼接
        filename: "bundle.js" //打包后的输出文件名
    },
    plugins: [
        new CleanWebpackPlugin({
            cleanOnceBeforeBuildPatterns: ['test/**/*', '!static-files*'],
        })
    ]
}
```

设置不清除的文件：

`cleanOnceBeforeBuildPatterns: ['test/**/*', '!test/a.js']`

这样就不会清除a.js文件了。

### 自动引入JS文件插件html-webpack-plugin

> https://www.npmjs.com/package/html-webpack-plugin
>
> 这个插件会把JS资源文件自动插入到HTML文件中，会在dist目录下输出新的转换后的HTML文件。

安装插件：

```
npm install html-webpack-plugin
```

在`webpack.config.js`文件下:

```js
const path = require("path")
// 引入清除目录插件，需要解构
const {
    CleanWebpackPlugin
} = require("clean-webpack-plugin")
// 引入自动插入JS文件插件，不用解构
const HtmlWebpackPlugin = require("html-webpack-plugin")
module.exports = {
    entry: "./src/index.js", //webpack的入口文件
    output: {
        path: path.resolve(__dirname, "dist"), //需要写绝对路径，用到node的全局变量__dirname(当前文件所在的路径)，与我们指定的目录dist拼接
        filename: "bundle.js" //打包后的输出文件名
    },
    plugins: [
        new CleanWebpackPlugin(
            // {
            // cleanOnceBeforeBuildPatterns: ['test/**/*', '!test/a.js'],
            // }
        ),
        new HtmlWebpackPlugin({
            template: "./index.html", //模板静态资源文件
            filename: "home.html", //输出的文件名
            hash: true, //添加哈希值，清除缓存
            minify: {
                removeAttributeQuotes: true, //去掉属性的引号
                collapseWhitespace: true, //去掉空格
                removeComments: true //去掉注释
            },
          inject:"body"//在哪插入JS文件
        })

    ]
}
```

> 更多的minify选项可以看这里：
>
> https://www.cnblogs.com/YangJieCheng/p/8302975.html(中文)
>
> https://github.com/kangax/html-minifier#options-quick-reference

### 启动本地服务 webpack-dev-server

> 这个插件可以实现在内存中打包，并自动启动一个本地服务器。在dist目录下是不会有文件生成的。
>

安装：

```
npm install webpack-dev-server -D
```

在`package.json`中，添加一个开启服务的命令：

```json
"scripts": {
    "dev": "webpack --mode development ",
    "build": "webpack --mode production ",
    "dev:server": "webpack-dev-server"
  }
```

还可以在`webpack.config.js`中设置相关参数。[devServer文档][https://www.webpackjs.com/configuration/dev-server/]

```
module.exports = {
	...
	devServer: {
        port: 9999, //端口号
        open: true, //自动打开窗口
        compress: true, //开启gizp压缩
        hot: true, //css代码热更新，只支持style标签的css代码
        contentBase: path.resolve(__dirname, "dist") ////以dist为目录，访问该目录的静态资源
    },
   ...
}
```

开发环境的css代码一般打包在style标签里，开发完上线后一般会把css代码分离出来，通过link标签引入。

#### `devServer.contentBase`

> 告诉服务器从哪里提供内容。只有在你想要提供静态文件时才需要。默认情况下，将使用当前工作目录作为提供内容的目录，但是你可以修改为其他目录。注意，推荐使用绝对路径。

```
contentBase: path.resolve(__dirname, "dist")
```

此时打开的默认页面是空白的：

![image-20200925000640489](https://i.loli.net/2020/09/25/5p1IiNJsfSDnmyq.png)

可以在地址后面添加我们dist目录下的静态文件名：http://localhost:9999/home.html，注意：如果`new HtmlWebpackPlugin()`里面的filename是index.html 会默认解析成这个页面，而不是空白页面，因为它会从dist目录寻找index.html文件，如果没有则显示空白。正常情况是改成index,html，这里只是演示功能改成home.html。

![image-20200925001118258](https://i.loli.net/2020/09/25/UlZWwSHutNOVeq3.png)

如果我们想访问根目录下的静态资源文件，比如在根目录新建一个other.html文件，我们访问时只需要把第二个参数“dist”去掉,访问的就是根目录了。

```
contentBase: path.resolve(__dirname)
```

打开http://localhost:9999/other.html

![image-20200925001642677](https://i.loli.net/2020/09/25/D1AGjahO3SiId4L.png)

### 多入口打包

> 配置多个入口文件，生成不同的JS文件，并且引入到不同的html文件中。

`webpack.config.js`文件下改成：

```js
  entry: {
        index: "./src/index.js",
        other: "./src/other.js"
    },
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "[name].js" //用文件自己的名字
    },
```

这样默认会把index.js和other.js都引入到index.html中，我们可以在`new HtmlWebpackPlugin`下修改成只引入我们想要的文件，添加`chunks: ["index"]`这个参数即可。

```js
new HtmlWebpackPlugin({
            template: "./index.html", //模板静态资源文件
            filename: "home.html", //输出的文件名
            hash: true, //添加哈希值，清除缓存
            minify: {
                removeAttributeQuotes: true, //去掉属性的引号
                collapseWhitespace: true, //去掉空格
                removeComments: true //去掉注释
            },
            inject: "body", //在哪插入JS文件
            chunks: ["index"] //多入口设置引入的文件，表示只引入index.js这个文件到index.html里，other.js不引入
        })
```

#### 需求：多个JS文件引入到多个html文件中

> 实际项目中，我们可能会有多个页面，需要把不同的JS资源引入到不同的页面中。

​		我们只需要多创建一个`HtmlWebpackPlugin`对象即可实现.

```js
plugins: [
        new HtmlWebpackPlugin({
            template: "./index.html", //模板静态资源文件
            filename: "home.html", //输出的文件名
            inject: "body", //在哪插入JS文件
            chunks: ["index"] //多入口设置引入的文件，表示只引入index.js这个文件到index.html里，other.js不引入
        }),
        // 实现多入口文件，不同JS文件插入到不同html文件，只需要新new一个即可
        new HtmlWebpackPlugin({
            template: "./other.html", //模板静态资源文件
            filename: "other.html", //输出的文件名
            inject: "body", //在哪插入JS文件
            chunks: ["other"] //只插入other.js文件
        })
    ]
```

#### 进阶操作：使用数组批量操作

> 如果有很多个入口文件，我们总不能一个个new把，这样代码太多，而且重复，我们修改的只是入口名字，使用数组结合map函数、展开运算符可以实现。

在`webpack.config.js`的开头定义一个变量：

```js
// 多个入口文件，高级写法
const htmlPlugin = ["index", "other"].map((chunckName) => {
    return new HtmlWebpackPlugin({
        template: `./${chunckName}.html`, //模板静态资源文件
        filename: `${chunckName}.html`, //输出的文件名
        inject: "body", //false，代表不插入文件，写了就代表在哪插入JS文件
        chunksSortMode: "manual", //auto代表自动插入 manual代表按chunks:["other","index"]的自定义顺序插入，比如jquery要先引入，index.js在后面引入。
        chunks: [chunckName] //只插入chunckName.js文件
    })
})
```

在` plugins`里使用ES6的展开运算符即可实现了。

```js
 plugins:[...htmlPlugin]
```

- **inject** 设置false代表不引入文件
- **chunksSortMode**: "manual", //auto代表自动插入 manual代表按chunks:["other","index"]的自定义顺序插入，比如jquery要先引入，index.js在后面引入。如果不定义，就会自动插入，顺序可能会乱。