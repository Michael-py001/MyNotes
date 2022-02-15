## webpack第四天-loader&插件的编写

### loader的原理和实现

1. 设置loader的默认配置  lib/webpack.js 

   把用户的配置与webpack的默认配置进行合并

2. 通过resolveLoader 解析loader模块路径  node_modules

3. 根据rule.modules创建RulesSet规则集

4. 使用loader-runner运行loader

### 设置resolveLoader解析loader模块路径

> 在根目录下创建loader文件夹，里面新建三个loader-xxx.js文件。

在`webpack.config.js`中设置resolveLoader

#### 模块查找

> resolveLoader.modules 模块查找，对应的rules下的use.loader是指文件名

```js
const path = require("path");
module.exports = {
    mode: "development",
    entry: './src/index.js',
    output: {
        filename: 'bundle.js'
    },
    resolveLoader: {
        // 在modules中设置查找loader的优先级
        // 先查找node_modules下的，没有再查找指定文件夹下的
        modules: [path.resolve(__dirname, "node_modules"), path.resolve(__dirname, "loader")]
    },
    module: {
        rules: [  //从下往下，从左往右
            {
                test: /\.js$/,
                use: {
                    loader: "loader1"//此时指的是文件名
                }
            },
            {
                test: /\.js$/,
                use: {
                    loader: "loader2"
                }
            },
            {
                test: /\.js$/,
                use: {
                    loader: "loader3"
                }
            }

        ]
    }
}

```

#### 别名查找

> resolveLoader.alias 设置loader的别名，对应的rules下的use.loader指的是alias下设置的loader别名。

```js
 resolveLoader: {
   			//设置对应文件名的别名
         alias: {
             loader1: path.resolve(__dirname, "loader", "loader1.js"),
             loader2: path.resolve(__dirname, "loader", "loader2.js"),
             loader3: path.resolve(__dirname, "loader", "loader3.js")
         }
    }
```

运行`npx webpack` 可以输出3、2、1.

### loader.pitch

> loader.pitch是在解析Loader前执行的，优先级高于loader，每一个loader都有一个loader.pitch

#### pitch无返回值

![image-20201010180016453](https://i.loli.net/2020/10/10/93K6lE2OCihVoDT.png)

在每个loader.js下添加loader.pitch

```js
loader.pitch = function () {
    console.log("loader-pitch1")
}
```

输出的顺序的倒序的，跟loader解析的顺序相反

![image-20201010010638854](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201010010638854.png)

#### pitch有返回值

> pitch有返回值会中断loader的执行

![image-20201010180158536](https://i.loli.net/2020/10/10/ukNDfe5UJzv1CB6.png)

在Loader2.js中添加return "a"

```js
loader.pitch = function () {
    console.log("loader-pitch2")
    return "a"
}
```

输出结果：

![image-20201010180446359](https://i.loli.net/2020/10/10/fhKV19M4QRNa8GD.png)

### enforce参数

> enforce有"pre"、"post"参数
>
> enforce:"pre" -->优先加载
>
> enforce:"post" -->最后加载

```
module: {
        rules: [  //从下往下，从左往右
            {
                test: /\.js$/,
                use: {
                    loader: "loader1"
                },
                enforce: "pre"
            },
            {
                test: /\.js$/,
                use: {
                    loader: "loader2"
                }
            },
            {
                test: /\.js$/,
                use: {
                    loader: "loader3"
                },
                enforce: "post"
            }

        ]
    }
```

输出结果：

![image-20201010175743119](https://i.loli.net/2020/10/10/Watd5CzMpxmrOKc.png)

### loader的类型

> loader的类型：pre(优先)  normal(默认)  inline(第三)  post(最后)

上面我们已经使用了enforce设置了pre，post类型，接下来将尝试inline类型。

1. 在loader文件夹下新建一个js文件，名为`inline-loader.js`（名字随便起），代码如下：

   ```js
   function loader(source) {
       console.log("inline-loader");
       return source;
   }
   module.exports = loader;
   ```

2. 在src下新建`a.js`文件，随便写入一行代码。然后在`index.js`中引入`a.js`

```js
//index.js
//引入a.js 使用inline-loader.js 作为loader
require("inline-loader!./a");//inline-loader指的是文件名 a.js文件用inline-loader加载
```

输出顺序如下：

![image-20201010223001699](https://i.loli.net/2020/10/10/Cp5iEguWskOXDy7.png)

可以得出结论，在index.js中引入的loader即为inline类型的loader，输出顺序为第三。

### require中loader加载的参数

1. `!!`不要其他loader处理，只是inline-loader处理

2. `-!`不要执行当前inline-loader前面的loader

3. ` !` normal类型的loader不执行

## 实现简版less-loader

安装 less，注意：不要安装`less-loader`与`style-loader`，这里用的是自己写的loader。

```
npm install less -D
```

会把less里面的render方法，把解析后的css代码返回。

loader文件夹下新建`less-loader.js`、`style-loader.js`;

`less-loader.js`:

```js
let less = require("less");
function loader(source) {
    let css;
    //source：传进来的Less代码
    //err 错误信息
    //result：处理的结果
    less.render(source, (err, result) => {
        css = result.css;
    })
    return css;
}
module.exports = loader;
```

`style-loader.js`

```js
function loader(source) {
    //1. 先创建style标签 2. 把样式作为style的内容 3. 把style标签插入到head标签内
    console.log(source);
    let code = `
        let style = document.createElement("style");
        style.innerHTML = ${JSON.stringify(source)};
        document.head.appendChild(style)
    `
    return code;
}
module.exports = loader;
```

src下新建`index.less`：

```less
@color :red;
div{
    background: @color;
}
```

`index.js`引入.less文件：

```js
import "./index.less"
```

`webpack.config.js`设置rules:

```js
    module: {
        rules: [ //从下往上 从右往左
            {
                test: /\.less$/,
                use: ["style-loader", "less-loader"]
            }
        ]
    }
```

运行`npx webpack`，在dist目录下生成bundle.js，新建一个index.html文件引入bundle.js查看效果

![image-20201011013550986](https://i.loli.net/2020/10/11/E4i85D2aUk79tKs.png)

## 实现babel-loader

首先安装`@babel/core @babel/preset-env`

```
npm install @babel/core @babel/preset-env -D
```

src下新建`babel-loader`：

```js
let babel = require("@babel/core");
//转换es6代码时要通过babel.transform进行转换，并且还需要把options下的这个@babel/preset-env预设传进去
function loader(source) {
    //this是一个loaderContext对象，传进来的参数都是放在this.query下
    console.log(Object.keys(this));
    console.log(this.query);
    return source;
}
module.exports = loader;
```

`webpack.config.js`下设置rules:

```js
 module: {
        rules: [  //从下往下，从左往右
            // ES6转ES5
            {
                test: /\.js/,
                use: {
                    loader: "babel-loader",
                    options: {
                        presets: [
                            "@babel/preset-env"
                        ]
                    }
                }
            }
        ]
    }
```



### loader中的this

> 官方文档：https://webpack.docschina.org/api/loaders/
>
> loader中的this就是loader的上下文，包含很多属性，其中this.query指的是`webpack.config.js`中use的options包含的参数。

`console.log(Object.keys(this))`输出如下：

![image-20201011023846844](https://i.loli.net/2020/10/11/nVC49wZfvheBbAo.png)

`this.query`输出如下：

![image-20201011023923968](https://i.loli.net/2020/10/11/5xD9WNwkja2h3rU.png)

多加一个plugins:

![image-20201011024031484](https://i.loli.net/2020/10/11/4EPkV96QNimJsYX.png)

### loader-utils工具包

> loader-utils https://github.com/webpack/loader-utils
>
> 专门用来处理loader的options的工具包。相当于this,query

安装：

```js
npm install loader-utils -D
```

使用loader-utils：

```js
const babel = require("@babel/core");
const loaderUtils = require("loader-utils")
//转换es6代码时要通过babel.transform进行转换，并且还需要把options下的这个@babel/preset-env预设传进去
function loader(source) {
    const options = loaderUtils.getOptions(this);
    console.log(options);
    return source;
}
module.exports = loader;
```

输出的结果和this.query一致：

![image-20201011025113372](https://i.loli.net/2020/10/11/N6kypMbEYnxXPOV.png)

### 使用@babel/preset-env

`babel-loader`:

```js
const babel = require("@babel/core");
const loaderUtils = require("loader-utils")
//转换es6代码时要通过babel.transform进行转换，并且还需要把options下的这个@babel/preset-env预设传进去
function loader(source) {
    const options = loaderUtils.getOptions(this);
    let cb = this.async()//会把loader变成一个异步的loader，并且返回一个异步的回调。

    //第一个参数：传进来的代码
    //第二个参数：optons选项内容
    //第三个参数：回调函数
    babel.transform(source, {
        ...options  //展开运算符取出options的内容
    }, function (err, result) {
        cb(err, result.code) //result.code表示结果
    })

    return source;
}
module.exports = loader;
```

`index.js`下：

```js
const sum = (a, b) => {
    return a + b
}
console.log(sum(10, 20))
```

`webpack.config.js`:

```js
{
  test: /\.js/,
    use: {
      loader: "babel-loader",
        options: {
          presets: [
            "@babel/preset-env"
          ],
            plugins: []
        }
    }
}
```

npx webpack运行，输出结果：

![image-20201011033040719](https://i.loli.net/2020/10/11/aJYfrWmpeXRF9Iz.png)

成功把es6代码转换成es5.

### 加入代码调试工具sourceMap

`webpack.config.js`加入devtool：

```
devtool: "cheap-module-eval-source-map",//代码调试工具
```

`babel-loader.js`中，cb加入第三个参数：result.map

```js
const babel = require("@babel/core");
const loaderUtils = require("loader-utils")
//转换es6代码时要通过babel.transform进行转换，并且还需要把options下的这个@babel/preset-env预设传进去
function loader(source) {
    const options = loaderUtils.getOptions(this);
    let cb = this.async()//会把loader变成一个异步的loader，并且返回一个异步的回调。

    //第一个参数：传进来的代码
    //第二个参数：optons选项内容
    //第三个参数：回调函数
    babel.transform(source, {
        ...options,  //展开运算符取出options的内容
        sourceMaps: true, //代码调试
        filename: this.resourcePath //报错时，对应的哪个文件名
    }, function (err, result) {
      //加入第三个参数
        cb(err, result.code, result.map) //result.code表示结果
      //map指的是source-map产生源码的映射
    })

    return source;
}
module.exports = loader;
```

如果在index.js中有代码出错时，会显示具体的哪一行，哪个文件报错：

![image-20201011034204646](https://i.loli.net/2020/10/11/JNbvuCdpysExO4L.png)

## 实现一个记录编译后所有文件及大小的plugin

> 实现插件的效果：记录编译后所有文件的名字和文件的大小；
>
> 文件名    文件大小
>
> style.css     xxx kb
>
> bundle.js    xxx kb
>
> 文件总数：3个

新建plugin文件夹：

```
npm init -y  //初始化文件夹
//安装所需要的包
npm install webpack webpack-cli -D  
npm install css-loader style-loader html-webpack-plugin mini-css-extract-plugin -D
```

### 关于插件编写的重要点compiler 钩子与compilation 钩子

#### compiler 钩子

> compiler代表一个完整的webpack环境配置，在启动webpack时就已经生成了，包括loder,options, plugins。
>
> https://webpack.docschina.org/api/compiler-hooks/

#### compilation 钩子

> 当资源文件、状态、依赖信息发生变化时，会新生成一个compilation对象 。https://webpack.docschina.org/api/compilation-hooks/

新建一个plugin文件夹，里面新建一个my_plugin.js

```js
class MyPlugin {
    // 解构一下，防止有其他参数传入
    constructor({ filename }) {
        this.filename = filename;
    }
    // 运行apply方法，接收一个compiler参数，这样保证在回调函数中可以访问compoler对象
    apply(compiler) {
        compiler.hooks.emit.tap("MyPlugin", (compilation) => {
            console.log(compilation.assets)
            // compilation.assets的格式：
            /* 
                {
                    'main.css':{source:"xxx",size:"xxx"},
                    'bundle.js':{source:"xxx",size:"xxx"},

                }
                Object.entries()返回一个数组，其元素是与直接在object上找到的可枚举属性键值对相对应的数组。

                使用Object.entries()把它变成数组格式，易于操作：
                [
                    ['main.css',{source:"xxx",size:"xxx"}]
                    ['bundle.js',{source:"xxx",size:"xxx"}]
                ]
            */
            let assets = compilation.assets;
            let content = "# 文件名     文件大小";
            Object.entries(assets).forEach(([filename, fileObj]) => {
                // filename就是assets里面的每一个对象-->bundle.js
                //fileObj指的就是{source:"xxx",size:"xxx"}
                content += `\r\n - ${filename}   ${fileObj.size()}b`
            })
            content += `\r\n 文件总个数：   ${Object.entries(assets).length}个`
            compilation.assets[this.filename] = {
                source() {
                    return content
                },
                size() {
                    return content.length
                }
            }
        })
    }
}
module.exports = MyPlugin
```

Object.entries()返回一个数组，其元素是与直接在object上找到的可枚举属性键值对相对应的数组。