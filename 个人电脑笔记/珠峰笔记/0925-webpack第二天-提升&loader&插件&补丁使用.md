[TOC]



## 0925-webpack第二天-提升&loader&插件&补丁使用

### 安装环境

> 新建一个webpack_day2文件夹，安装第一天所需的模块

运行`npm install`就可以把package.json中开发环境和生产环境所需要的包都安装上。

### loader的使用

> webpack天生自带识别JS文件，但是CSS / 图片 / 高版本的JS语法（ES6 ES7...）是不能识别的，所以我们需要一个解释器去解析，这个解析器就是loader。
>
> 这里使用的是在`webapck.config.js`里配置loader

​	loader的位置是在`module`下的`rules`里面，可以包含多个loader，每一个loader放在一个对象当中。

```js
module.exports = {
  module: {
    rules: [
      { test: /\.css$/, use: 'css-loader' },
      { test: /\.ts$/, use: 'ts-loader' }
    ]
  }
};
```

#### loader包含的参数

- **test**    匹配处理文件扩展名的正则表达式。
- **use**     loader名称，就是你要使用的模块名称
- include/exclude     手动指定必须要处理的文件夹 /  屏蔽不需要处理的文件夹
- options     为loaders提供额外的设置选项

#### loader执行的顺序

​		默认loader的执行顺序：**从下往上 ， 从右往左** 。

当然执行的顺序也可以手动定义。

### 处理CSS文件

比如src目录下有sytle.css / index.js文件；

style.css:

```css
div {
    color: red;
}
```

idnex.js:

```js
import "./style.css"
```

默认情况下，webpack是不能识别css代码的，会报错：

![image-20200925213800773](https://i.loli.net/2020/09/25/j6GUsiqPlVxeZa3.png)

我们需要安装解析css代码的loader模块:`style-loader`、`css-loader`

```
npm install style-loader css-loader -D
```

#### 功能

- `css-loader`  解析CSS代码

- `style-loader`   把CSS代码放到style标签里

之后就可以在rules里编写我们的loader，记住loader的加载顺序是从下往上，从右往左。

现在我们需要解析的是CSS代码，loader的加载顺序是：先加载`css-loader`，后加载`style-loader`，两者的顺序不能错，不然webpack会报错。

**而如果我们想指定哪个loader先加载/后加载该怎么实现呢？**

#### 方式一：加入 enforce参数

> **enforce:  "pre(先加载) / post(后加载)"**

```js
rules: [ //从下往上，从右往左
  {
    test: /\.css$/,
    use: "css-loader"
  },
  {
    test: /\.css$/,
    use: "style-loader",
    enforce: "post" //pre先加载  post后加载
  }
]
```

这样webpack就能正常运行，字体显示为红色了。

![image-20200925222537608](https://i.loli.net/2020/09/25/8uXhCtkd53RqfiV.png)

#### 方式二：数组中排序

```js
 rules: [ //从下往上，从右往左
   {
   test: /\.css$/,
   use: ["style-loader", "css-loader"]//从右往左加载
   }

 ]
```

### 处理CSS预处理器

- 解析less  需要安装的包：less  less-loader
- 解析sass  需要安装的包：node-sadd  sass-loader
- 解析stylus  需要安装的包：stylus  stylus-loader

这里我们测试**less**的处理。

- less  解析less代码的核心包，这个不用在loader里导入
- less-loader  webpack里Loader的解析包

#### 情况一：JS中导入less

创建一个style.less文件：

```less
@color: green;

div {
    background: @color;
}
```

index.js文件引入less文件

```js
import "./style.less"
```

rules:

```js
  rules: [ //从下往上，从右往左
    {
    test: /\.less$/,
    use: ["style-loader", "css-loader", "less-loader"]
    }
  ]
```

正常显示：

![image-20200925224959746](https://i.loli.net/2020/09/25/xzBtlumA2KNQjeR.png)

#### 情况二：CSS引入less，JS引入CSS

style.css:

```js
@import "./style.less";
div {
    color: red;
}
```

index.js

```js
import "./style.css"
```

这种情况下，在CSS文件中引入了less文件，之前的两种loader配置都是行不通的，我们可以添加option参数，在处理CSS时按需加载less-loader。

rules:

```js
rules: [ //从下往上，从右往左
  {
    test: /\.css$/,
    use: ["style-loader",
      {
        loader: "css-loader",
        options: {
          importLoaders: 1 //需要后面的1个加载器
        }
      }, "less-loader" //这里就是option需要的加载器
    ]
  },
  {
    test: /\.less$/,
    use: ["style-loader", "css-loader", "less-loader"]
	}
]
```

这样就能正常显示了。

### 处理css3私有前缀(各个浏览器兼容前缀)

**postcss-loader**

> PostCSS 本身是一个功能比较单一的工具。它提供了一种方式用 JavaScript 代码来处理 CSS。它负责把 CSS 代码解析成抽象语法树结构（Abstract Syntax Tree，AST），再交由插件来进行处理。插件基于 CSS 代码的 AST 所能进行的操作是多种多样的，比如可以支持变量和混入（mixin），增加浏览器相关的声明前缀，或是把使用将来的 CSS 规范的样式规则转译（transpile）成当前的 CSS 规范支持的格式。[https://my.oschina.net/kaykie/blog/911131][https://my.oschina.net/kaykie/blog/911131]

- postcss-loader(样式处理器，仅仅是处理样式的，其他用法请看官网)[https://www.npmjs.com/package/postcss-loader][https://www.npmjs.com/package/postcss-loader]
- autoprefixer(增加私有前缀的插件)

安装：

```
npm install postcss-loader autoprefixer@8 -D (注意是8版本)
```

rules:

```js
 rules: [ //从下往上，从右往左
   {
     test: /\.css$/,
     use: ["style-loader",
           {
             loader: "css-loader",
             options: {
               importLoaders: 2 //需要后面的2个加载器
             }
           }, "postcss-loader", "less-loader" //这里就是option需要的加载器
          ]
   },
   {
     test: /\.less$/,
     use: ["style-loader", "css-loader", "less-loader"]
   }
 ]
```

现在postcss-loader已经能够使用了，但是实现增加私有前缀的功能需要继续添加配置。

根目录下新建一个`postcss.config.js`文件：

```js
module.exports = {
    plugins: [
        require("autoprefixer")
    ]
}
```

这样在加载postcss-loader时，就会自动去查找postcss.config.js这个文件，然后执行功能。

如果需要加的前缀兼容大部分浏览器，则需要再建一个`.browserslistrc`文件，写入以下内容：

```
cover 99.9%
```

这样就会添加多个浏览器私有前缀了

![image-20200925235541622](https://i.loli.net/2020/09/25/4PqtvsGaHhenU7f.png)

### 抽离CSS文件

> 项目上线时，一般是把css文件抽离出来，通过Link的方式导入到html中。

需要安装的插件：`mini-css-extract-plugin`

```
npm install mini-css-extract-plugin -D
```

在`webpack.base.js`中引入插件

```js
// 引入抽离CSS代码的插件
const MiniCssExtracPlugin = require("mini-css-extract-plugin")
```

module.exports里，把env抽离出来设置成一个变量

```js
let isDev = env.development //把环境设置成一个变量
```

rules:

```js
 rules: [ //从下往上，从右往左
                {
                    test: /\.css$/,
                    use: [ //"style-loader", 引入MiniCssExtracPlugin后就不用添加到style标签上了，直接通过外链的方式导入
                        {
                            loader: isDev ? "style-loader" : MiniCssExtracPlugin.loader  //判断哪个环境用哪个
                        },
                        {
                            loader: "css-loader",
                            options: {
                                importLoaders: 2 //需要后面的2个加载器
                            }
                        }, "postcss-loader", "less-loader" //这里就是option需要的加载器
                    ]
                },
                {
                    test: /\.less$/,
                    use: ["style-loader", "css-loader", "less-loader"]
                }
            ]
```

plusgins:

```js
 plugins: [
            !isDev && new MiniCssExtracPlugin({
                filename: "css/[name].[contentHash:4].css" //contentHash 根据每个文件的内容产生的hash值
            }), //isDev为true才执行这个插件
            new CleanWebpackPlugin(),
            new HtmlWebpackPlugin({
                template: "./index.html", //模板静态资源文件
                filename: "index.html", //输出的文件名

            })
        ].filter(Boolean) //过滤布尔值，因为开发环境下!isDev会返回false，而plugins数组中是不能有布尔值的
```

这样就能实现抽离CSS文件了

![image-20200926003408953](https://i.loli.net/2020/09/26/75FJUVlnMCdGgO6.png)

#### 文件指纹

- Hash:整个项目的Hash值
- chunkHash:根据入口产生的hash值
- contentHash:根据每个文件的内容产生的hash值

上面使用了contentHash，后面可以指定长度。

### CSS/JS文件压缩

CSS压缩插件包名：`optimize-css-assets-webpack-plugin`

JS压缩插件包名：`terser-webpack-plugin`

> 这里需要注意的是，本来生产环境下运行npm run build会自动压缩JS的，但是用了CSS压缩后，JS代码就不压缩了，所以得CSS 和JS一起压缩。

安装压缩CSS插件：

```
npm install optimize-css-assets-webpack-plugin -D
```

安装压缩JS插件：

```
npm install terser-webpack-plugin -D
```

在`webpack.prod.js`文件中配置：

```js
// 压缩CSS代码的插件
const OptimizeCssAssetsPlugin = require("optimize-css-assets-webpack-plugin")
// 压缩JS代码的插件
const TerserPlugin = require("terser-webpack-plugin")
module.exports = {
    mode: "production", //添加环境 development production none 三者其一
    optimization: { //optimization优化的意思
        minimize: true,
        minimizer: [new OptimizeCssAssetsPlugin(), new TerserPlugin()] //把CSS/JS压缩的插件都引进来
    },
}
```

最后效果是CSS/JS都压缩了：

![image-20200926010409157](https://i.loli.net/2020/09/26/HbvI35B1CnZmsAk.png)

![image-20200926010424834](https://i.loli.net/2020/09/26/R2sCZ5dBi6Ov1PE.png)

### 解析图片

- file-loader 会将图片进行打包，并且讲打包后的图片返回 拷贝图片
- url-loader  当图片小于limit的时候会将图片转换成BASET64编码，大于limit参数时会使用file-loader 进行拷贝

安装：

```
npm install file-loader url-loader -D
```

#### file-loader的使用

`index.js`文件：

```js
import "./style.css";
import url from "./images/1.png";
let img = document.createElement("img");
img.src = url;
document.body.appendChild(img)
```

`loader`中的设置：

```js
    {
      test: /\.(png|gif|jpe?g)$/,
        use: {
          loader: "file-loader",
            // 设置图片名称，不写默认生成名字
            options: {
              name: "img/[name].[ext]"
            }
        }
    }
```

完成后，在style.css文件中设置div背景图片也是可以解析到的。

```css
div {
    color: red;
    width: 100px;
    height: 100px;
    transform: rotate(45deg);
    background: url("./images/1.png")
}
```

![image-20200926013943405](https://i.loli.net/2020/09/26/amiEL8H2qX5PvGA.png)

#### url-loader 的使用

loader设置与file-loader差不多：

```js
 // url-loader的使用
                {
                    test: /\.(png|gif|jpe?g)$/,
                    use: {
                        loader: "url-loader",
                        // 设置图片名称，不写默认生成名字
                        options: {
                            limit: 300 * 1024, //图片小于这个值，则用base64转码，大于则用file-loader进行拷贝地址
                            name: "img/[name].[ext]"
                        }
                    }
                }
```

- [name] 与[ext] 都是webpack中自带的全局变量，代表原来文件的名字

![image-20200926014810314](https://i.loli.net/2020/09/26/xya8zPoDGYF6Egn.png)

### 处理html里的Img图片

​	如果我们的index.html文件里有一个img标签，那么以上的方案都满足不了我们的需求了，这时候需要安装一个`html-withimg-loader`插件。

如果你设置了以下路径的图片，在开启devServer预览的时候可能会正常显示图片，原因是你的webpack.dev.js文件下devServer配置中的contentBase的路径可能是：`path.resolve(__dirname)`以根目录下为目录加载资源文件，但是改成其他路径时就会看不到了。

```
<img src="./src/images/1.png" alt="">
```

安装：

```
npm install html-withimg-loader -D
```

loader配置：

```js
  // html图片处理插件
                {
                    test: /\.(html|htm)$/,
                    use: "html-withimg-loader"
                }
```

这里会出现一个问题：图片的路径会出现`default`导致加载图片失败。

![image-20200926020541932](https://i.loli.net/2020/09/26/kAWmQ2OrJuR8nYM.png)

解决方案是：在url-loader里设置`esModule: false`,不按照模块导出

```js
  // url-loader的使用
                {
                    test: /\.(png|gif|jpe?g)$/,
                    use: {
                        loader: "url-loader",
                        // 设置图片名称，不写默认生成名字
                        options: {
                            esModule: false, //不按照模块导出
                            limit: 300 * 1024, //图片小于这个值，则用base64转码，大于则用file-loader进行拷贝地址
                            name: "img/[name].[ext]"
                        }
                    }
                }
```

### 处理icon图标

> 从阿里巴巴图标库下载的图片引入到html需要用到`file-loader`模块解析

index.js文件引入图标：

```js
import "./style.css";
import "./icon/iconfont.css";
let i = document.createElement("i");
i.className = "iconfont icon-kehuishouwu";
i.style.setProperty("font-size", "130px")
document.body.appendChild(i);
```

loader添加一个`file-loader`

```js
 // 处理字体图标
{
  test: /\.(eot|svg|ttf|woff|woff2)/,//匹配这几种后缀
    use: "file-loader"
}
```

效果：

![image-20200926093421703](https://i.loli.net/2020/09/26/zp2HlG4giudLE3f.png)

### 处理JS模块

> webpack处理ES6语法的JS代码会直接输出，不会转换成ES5，如果需要转换，我们需要用到babel模块。

#### 把ES6代码转换成ES5代码-->babel模块

需要安装的包：

- `@babel/core`  babel的核心模块
- `@babel/preset-env`  ES6代码转换成ES5插件的集合
- `babel-loader`  webpack和babel的桥梁

```
npm isntall @babel/core @babel/preset-env babel-loader -D
```

根目录新建一个`.babelrc`文件，在里面写入：

```json
{
    "presets": [
        "@babel/preset-env"
    ],
    "plugins": []
}
```

`webpack.base.js`配置loader

```js
  // babel转换ES6代码
{
  test: /\.js$/,
  use: "babel-loader",
  exclude: /node_modules/
}
```

转换后的效果：

![image-20200926221714149](https://i.loli.net/2020/09/26/DO6ge4SWlrzqIbN.png)

#### ES7语法转换补丁-->core-js

> 一些ES7的语法，用babel基本的预设是转换不了的，需要用专门的补丁转换。

在index.js里添加代码：

```js
import sum from "./sum"
document.write(sum(8, 8))
let a = Object.assign({}, {
    "a": 1
}) //es7
console.log(a);
[1, 2, 3].includes(1); //es7
async function fn() {
    console.log("ok")
}
fn()
```

不添加补丁前，ES7的语法并没有转换：

![image-20200926223452927](https://i.loli.net/2020/09/26/HegKxBtcYEICU8p.png)

安装`core-js`补丁：

```
npm install core-js -D
```

安装好后，在`.babelrc`下添加补丁配置：

```json
{
    "presets": [
        ["@babel/preset-env", {
            "useBuiltIns": "usage",
            "corejs": 3
        }]
    ],
    "plugins": []
}
```

打包后JS代码里显示用了core-js里的assign文件：![image-20200926224806344](https://i.loli.net/2020/09/26/aoDzScv2QtEfUAm.png)

重写了`includes`方法：

![image-20200926224845578](https://i.loli.net/2020/09/26/sEGW3B9FnwfSkjc.png)

async语法：

![image-20200926225147726](https://i.loli.net/2020/09/26/Waj1UVFBCMAtzJd.png)

#### 支持类语法的补丁

> 上面的core-js还不能支持类的语法，需要再安装一个补丁

index.js:

```js
class People {
    a = 1
}
new People()
```

运行后会报错：提示安装`@babel/plugin-proposal-class-properties`

![image-20200926225537907](https://i.loli.net/2020/09/26/QKbkiMROB6v4Ynw.png)

安装：

```
npm install @babel/plugin-proposal-class-properties -D
```

@babel/plugin-proposal-class-properties这是一个插件，放在plugins里面：

```json
//.babelrc
{
    "presets": [
        ["@babel/preset-env", {
            "useBuiltIns": "usage", //按需加载
            "corejs": 3 //core-js的版本
        }]
    ],
    "plugins": [
        "@babel/plugin-proposal-class-properties"
    ]
}
```

安装后转换的效果：

![image-20200926225855891](https://i.loli.net/2020/09/26/U2feywFDAM7STqk.png)

#### 转换还是草案的语法（装饰器）

> 包名：@babel/plugin-proposal-decorators
>
> https://babeljs.io/docs/en/babel-plugin-proposal-decorators

安装：

```
npm install @babel/plugin-proposal-decorators -D
```

`.babelrc.js`配置：

```json
{
    "presets": [
        ["@babel/preset-env", {
            "useBuiltIns": "usage", //按需加载
            "corejs": 3 //core-js的版本
        }]
    ],
    "plugins": [
        //"@babel/plugin-proposal-class-properties",
        ["@babel/plugin-proposal-decorators", { //处理装饰器的包
            "legacy": true //表示保留装饰器
        }],
        ["@babel/plugin-proposal-class-properties", { //处理类的包
            "loose": true //表示宽松语法方式，比如this.a = 1是宽松的，严格的就是用Object.definePropoty
        }]
    ]
}
```

转换效果：

![image-20200926231109449](https://i.loli.net/2020/09/26/DZUQrksIxKSdHGh.png)

如果配置里的`loose`设置为false，则会用：Object.defineProperty代替this.xxx设置属性。

![image-20200926231333210](https://i.loli.net/2020/09/26/Ir3FpUgE8lkvzC7.png)

### 优化打包

可以提升构建的速度，减少冗余的代码

#### @babel/plugin-transform-runtime && @babel/runtime（依赖包）

> 这两个包可以把公共方法提取出来，在每个模块使用时，调取公共方法即可，减少了代码冗余。

安装：

```
npm install @babel/plugin-transform-runtime @babel/runtime -D
```

使用：

在`.babelrc`:

```js
 "plugins": [
        "@babel/plugin-transform-runtime", //优化包
        ["@babel/plugin-proposal-decorators", { //处理装饰器的包
            "legacy": true //表示保留装饰器
        }],
        ["@babel/plugin-proposal-class-properties", { //处理类的包
            "loose": true //表示宽松语法方式，比如this.a = 1是宽松的，严格的就是用Object.defineProperty
        }]
    ]
```

使用后代码量减少了，从原来的1200多行减少成900多行：

![image-20200928154920992](https://i.loli.net/2020/09/28/GimnVH1WlwN6RFM.png)