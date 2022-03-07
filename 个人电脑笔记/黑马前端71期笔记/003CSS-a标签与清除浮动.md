## 目录

[TOC]

### a标签

```css
a:link{
	color: red;  /*默认字体颜色*/
}
a:visited{
	color: green;/*访问过的颜色*/
}
a:hover{
	color: blue;/*鼠标移上去时的颜色*/
}
a:active{
	color: yellow;/*鼠标点击下去时颜色*/
}
```

### 清除浮动的五种方法





#### 1. height

> 通过给父盒子添加高度，实现去除浮动。缺点就是要给父盒子添加高度，因为一般我们不给父盒子添加高度，所以这点不太好。



html结构如下
```html
<div class="box">
    <p></p>
    <p></p>
    <p></p>
    <p></p>
</div>
<div class="box">
    <p></p>
    <p></p>
    <p></p>
    <p></p>
</div>
```

没有高度时

![image-20200721001402908](https://i.loli.net/2020/07/21/XAJRK71MBjuUkfh.png)

父盒子会塌陷

![](https://i.loli.net/2020/07/21/ueRIdBA7FSOnNoX.png)

添加高度后

![image-20200721001753773](https://i.loli.net/2020/07/21/wyOZkG7bdUaDhQ8.png)

已经没有塌陷，实现了清除浮动

![image-20200721001839827](https://i.loli.net/2020/07/21/C49IonwYVRvFckt.png)

#### 2. clear

> clear:both 是指清除左浮动和右浮动，clear是指清除外界浮动对自身的影响。

html结构如下

![image-20200721004145762](https://i.loli.net/2020/07/21/f8Trv7zuagyHlNb.png)

css如下：

![image-20200721004414896](https://i.loli.net/2020/07/21/Cifd5j6ehWpR1b3.png)

效果：

![image-20200721004449711](https://i.loli.net/2020/07/21/IT3xykLwSvFbXfs.png)

> 缺点就是父盒子高度不能自适应。

#### 3. 外墙法

> 通过在两个div元素之间加一个div，把两个div隔离开来。

html结构：

![image-20200721004846862](https://i.loli.net/2020/07/21/7j9wgS3odcZaMhx.png)

css:

![image-20200721004933518](https://i.loli.net/2021/03/31/MNgRPjvu1wnrlTJ.png)

效果：

![image-20200721005112569](https://i.loli.net/2020/07/21/tAnF9NqsulWRibK.png)

> 中间的div添加clear:both; height:20px。缺点是父盒子高度还是不能 自适应。

#### 4. 内墙法

> 通过在每个浮动元素的最后添加一个子元素，实现清除浮动，而且父盒子高度可以自适应。

html:

![image-20200721005259914](https://i.loli.net/2020/07/21/LXjOZacEiyf1t3k.png)

css:

![image-20200721005318865](https://i.loli.net/2020/07/21/laeT8XGu4AjF2VE.png)

效果：

![image-20200721005422525](https://i.loli.net/2020/07/21/hepcZSEyx7qfiBw.png)

> 添加的子元素设置属性 clear:both 即可。
>
> 缺点：增加代码量，破坏原来的结构。

#### 5.溢出隐藏法（overflow:hidden)

> 通过给父盒子增加overflow:hidden的属性，实现清除浮动，原理就是浏览器在加载时会检测每个浮动元素的高度，取最高值为height撑起父盒子，而且会管理浮动元素不去影响外界的元素。

html:

![image-20200721010712973](https://i.loli.net/2020/07/21/wxUsWuKMhQpJDIV.png)



css:

![image-20200721010726648](https://i.loli.net/2020/07/21/V4FMCdwynhIexNi.png)

效果：

![image-20200721010752961](https://i.loli.net/2020/07/21/3IHUw4cJ8FAaqrt.png)

> 此方法推荐使用，只需要在父元素中添加overflow:hidden即可，不改变原来的结构，是个偏方，但是实用。

#### 6. 添加伪类

```css
.clearfix:after {
    content: ".";
    display: block;
    height: 0;
    clear: both;
    visibility: hidden;
}
/*一般用三个属性即可*/
.clearfix:after {
    content: "";
    display: block;
    clear: both;
}
.clearfix {
	zoom:1  /*兼容ie6*/
}
```

