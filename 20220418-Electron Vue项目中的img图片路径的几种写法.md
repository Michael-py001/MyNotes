# 20220418-Electron Vue项目中的img图片路径的几种写法.md

## webpack设置目录别名

在`vue.config.js`中:

```js
chainWebpack: (config) => {
    config.resolve.alias
        .set('@', resolve('src'))
        .set('assets', resolve('src/assets'))
        .set('@/components', resolve('src/components'))
        .set('@/utils', resolve('src/utils'))
},
```

这样设置完后，图片放在`src/assets/images`下，地址直接写`@/assets/images`即可

以上设置好后，开始下面的内容

## 普通Vue项目中的图片路径

- 如果图片放在src目录下，需要使用`@`拼接路径，这样写的路径会被webpack解析成正确的路径

```html
<img src="@/assets/images/bg.png">

```

- 如果把图片放在`public`目录下，可以使用绝对路径，需要注意的是放在public目录下的文件不会被webpack处理(压缩，base64)

​		比如该图片放在`public/images`下，打包编译后会生成在根目录`/images`下

```html
<img src="/images/bg.png">
```

## electron中的图片路径

上面的两种方式都能使用，但是有一种，使用变量的路径，不能够正确解析

比如：

```html
<img :src="imgPath">
```



## 解决办法

- 使用`require`引入地址必须使用静态的字符串，不能使用变量，因为require是编译时执行的，而非运行时执行！

正确：

```js
var imgUrl = require('../images/001.png');
```

报错

```js
var ImgSrc = "../images/001.png";
var img = require(ImgSrc)

require(`../../assets/images/${showAllExpended?'unfold':'up'}.png`
```

require里的正确的格式必须是path，在打包时require会根据path获取最终的路径，而如果是变量的话，只是传了变量名字进去，所以编译会报错

![img](https://s2.loli.net/2022/04/28/wa3KZCNoikR9FQz.png)

- 自己补全路径

  ```js
  imgPath : path.join(process.env.BASE_URL, 'logo.png')
  ```

  