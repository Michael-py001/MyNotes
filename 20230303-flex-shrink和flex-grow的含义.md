# 20230303-flex-shrink和flex-grow的含义

CSS中的`flex-grow`和`flex-shrink`是控制Flexbox布局中项目的伸缩性的属性。

- flex-grow：指定弹性盒子的空间分配策略，用来设置弹性容器内项目的放大比例。默认值为0，即不放大。
- flex-shrink：指定弹性盒子的空间分配策略，用来设置弹性容器内项目的缩小比例。默认值为1，即缩小。

flex-grow和flex-shrink均设置为0时，弹性容器和项目都不会按照这些属性变化尺寸。

在Flexbox布局中，项目的伸缩性可以通过以下方式进行定义：

```css
/* 指定所有项目的放大比例和缩小比例 */
flex: <flex-grow> <flex-shrink> <flex-basis>;

/* 分别指定项目的放大比例、缩小比例和初始大小 */
flex-grow: <number>; // 默认值为 0
flex-shrink: <number>; // 默认值为 1
flex-basis: <length> | auto; // 默认值为 auto

```

## flex-grow案例

以下代码设置一个包含3个项目的Flexbox布局，其中第一个项目的放大比例为2，第二个为1，第三个为3，缩小比例均为1：

```html
<div class="container">
  <div class="item-1">

  </div>
  <div class="item-2">

  </div>
  <div class="item-3">

  </div>
</div>
```

```less
.container {
    display: flex;
    justify-content: space-around;
    width: 500px;
    height: 200px;
}

.item-1 {
    flex-grow: 2;
    background-color: lightcoral;
    
}

.item-2 {
    flex-grow: 1;
    background-color: rgb(150, 240, 128);
}

.item-3 {
    flex-grow: 3;
    background-color: rgb(119, 106, 236);
}
```

![image-20230303183544187](https://s2.loli.net/2023/03/03/SFZOPpLXqv3zJRy.png)

该代码用于设置Flexbox容器的justify-content属性为space-around，用于在容器中平均分配空间。因此在空间容许的情况下，第三个项目将比第二个项目大3倍。

## flex-shrink案例

flex-shrink是CSS Flexbox布局中的一个属性，用于控制 flex元素在容器中的大小和分布情况。它的作用是在某些情况下，当flex元素的总宽度超出了容器的宽度时，它会根据设置的属性值缩小以适应容器的大小。

```html
<!-- 容器包含三个子元素 A、B、C -->
  <div class="container">
    <div class="a"></div>
    <div class="b"></div>
    <div class="c"></div>
  </div>
```

```css
 /* 定义 flex 属性 */
  .container {
    display: flex;
    width: 300px;
    border: 2px solid black;
  }

  /* 三个 flex 子元素，宽度各为 100px */
  .a {
    flex-shrink: 2;
    width: 100px;
    height: 100px;
    background-color: orange;
  }

  .b {
    flex-shrink: 1;
    width: 100px;
    height: 100px;
    background-color: pink;
  }

  .c {
    flex-shrink: 1;
    width: 100px;
    height: 100px;
    background-color: yellow;
  }
```

在上面的代码中，元素A具有最高的flex-shrink属性值，因此，如果容器的宽度发生变化，元素A将以两倍的速度缩小，而其他两个元素将以相同的速度缩小。



![image-20230303184922968](https://s2.loli.net/2023/03/03/eUqWsNJ4EiC9dMg.png)

如果容器的宽度被压缩到150像素，则A元素的宽度将被缩小到50像素，而B和C元素将被缩小到75像素。

![image-20230303184940527](https://s2.loli.net/2023/03/03/L5c67y3jDmUw9kE.png)