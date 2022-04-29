# 20220418-Electron项目中图片地址为何要用require引入?



## 普通Vue项目中引入图片的方式

- 如果图片放在src目录下，需要使用`@`拼接路径

```html
<img src="@/assets/images/bg.png">

```

- 如果把图片放在`public`目录下，可以使用绝对路径，需要注意的是放在public目录下的文件不会被webpack处理(压缩，base64)

  ```html
  <img src="/public/images/bg.png">
  ```

但是在Electron中使用Vue写的项目，以上的引入方法并不会成功引入，原因是在electron编译后，整个项目并不是在根目录，而是在electron内部文件的某个地方，所以用上面的方法引入的地址没办法编译成最终正确的地址。

## 解决办法

使用`require`引入地址必须使用静态的字符串，不能使用变量，因为require是编译时执行的，而非运行时执行！

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