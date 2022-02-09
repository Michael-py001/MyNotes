# 20211130-Vue项目安装sass-loader报错this.getOption is not a function

该项目是个老项目，Webpack还是3.6.0版本，现在想实现同时可以用`less`、`Scss/sass`

## 安装less

要用less,只需要安装两个包:

```
yarn add less less-loader
//我目前用的版本
yarn add less@3.10.3 less-loader@5.0.0
```

## 安装sass

node-sass 把 sass编译成css
sass-loader 是webpack的一个loader

老项目如果使用太高版本的sass-loader会出现this.getOption is not a function报错，需要安装低版本

node-sass是可以把scss/sass转换成

```
yarn add node-sass@4.14.1 @sass-loader@7.3.1
```

## webpack配置使用less sass

```js
module.exports = {
	module:{
		rules:[
			{
                test: /\.less$/,
                loader: "style-loader!css-loader!less-loader",
            },
            {
                test: /\.s[ac]ss$/,
                loader: "style-loader!css-loader!scss-loader",
            },
		]
	}
}
```

`style-loader!css-loader!less-loader`这是一种内联写法，官网中不推荐使用

下面是标准写法

```js
rules: [
      {
        test: /.less$/,
        use: [
          {
            loader: "style-loader",
            options:{},
            enforce:'post'
            // creates style nodes from JS strings
          },
          {
            loader: "css-loader"
            // translates CSS into CommonJS
          },
          {
            loader: "less-loader"
            // compiles Less to CSS
          }
        ]
      }
    ]
```

enforce是用来控制loader的执行顺序的

在这里可以把loader分为四种 pre normal inline post 其执行顺序 pre -> normal -> inline ->post，如:`enforce:'post'`代表最后执行