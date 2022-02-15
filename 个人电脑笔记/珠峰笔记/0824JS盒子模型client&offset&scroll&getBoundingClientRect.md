### 目录

[TOC]



### JS的盒子模型

> 基于一系列的属性方法获取元素的一些样式

#### node.getBoundingClientRect()

> 返回该元素的width,height,top,bottom,left,right,x,y;
>
> 除了`width` 和 `height` 以外的属性是相对于**视图窗口**(浏览器窗口)的左上角来计算的。
>
> var rect = obj.getBoundingClientRect();

<img src="https://i.loli.net/2020/08/24/T3XfmcK781Zvz6s.png" alt="DOMRect 示例图" style="zoom: 50%;" />

![image-20200824020138920](https://i.loli.net/2020/08/24/4tjrvXWlfdizYMQ.png)

#### cilent系列

##### clientWidth/cilentHeight

> 返回内容的可视宽度/高度，该属性包括内边距 padding，但不包括边框 border、外边距 margin 和垂直滚动条

![Image:Dimensions-client.png](https://i.loli.net/2020/08/24/LxZhwpBjDH4rnt8.png)

##### clientTop/clientLeft

> 返回div的**上边框**/**左边框**的大小,其值为一个整数，没有单位

![在这里插入图片描述](https://i.loli.net/2020/08/24/4TidxLkUYwPQKWX.png)

##### clientX/clientY

> clientX/Y 事件属性返回当事件被触发时鼠标指针相对于浏览器页面（或客户区）的水平/垂直坐标。
>
> 客户区指的是当前窗口。

#### offset系列

##### offsetWidth/offsetHeight

> 返回元素的宽度/高度，包含元素的边框(border)、水平线上的内边距(padding)、竖直方向滚动条(scrollbar)

##### offsetTop

> 返回当前元素相对于其 [`offsetParent`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLElement/offsetParent) 元素的顶部内边距的距离。

##### offsetLeft

> 返回当前元素*左上角*相对于  [`HTMLElement.offsetParent`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLElement/offsetParent) 节点的左边界偏移的像素值

##### offsetParent

![img](https://i.loli.net/2020/08/24/hwbNxYSFM49527U.jpg)

**与当前元素最近的经过定位(position不等于static)的父级元素**

【1】元素自身有fixed定位，offsetParent的结果为null

【2】元素自身无fixed定位，且父级元素都未经过定位，offsetParent的结果为<body>

【3】元素自身无fixed定位，且父级元素存在经过定位的元素，offsetParent的结果为离自身元素最近的经过定位的父级元素

【4】<body>元素的parentNode是null

#### scroll系列

##### scrollWidth/scrollHeight

- 没有内容溢出的情况下和clientWidth/clientHeight是相等的

  ![img](https://i.loli.net/2020/08/24/NQIGMKjRPnDE3rz.png)

- 有内容溢出的情况下要包含溢出的内容（真实的内容大小）

  ![img](https://i.loli.net/2020/08/24/QxU6ugzGLOV2yBi.png)

##### scrollTop/scrollLeft

> 获取滚动条卷去的宽度或者高度,只有这两个属性是“可读写”的：可以设置值;
>
> 这个元素的**内容顶部**（卷起来的）到它的视口可见内容（的顶部）的距离的度量。当一个元素的内容没有产生垂直方向的滚动条，那么它的 `scrollTop` 值为`0`。

- 设置HTML.scrollTop=0直接回到顶部