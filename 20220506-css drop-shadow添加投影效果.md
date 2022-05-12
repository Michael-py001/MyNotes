# 20220506-css drop-shadow添加投影效果

通常我们用box-shadow来设置盒阴影，但是顾名思义，盒阴影一般都是矩形，如果想要做其他形状，比如三角形，该怎么做？

答案是使用`drop-shadow`

`drop-shadow`滤镜可以给元素或图片非透明区域添加投影。

投影 vs 盒阴影
实践出真知，下面我们用CSS border写一个虚线框，例如：

```css
border: 10px dashed #beceeb;
```

结果长相如下：

然后，我们分别应用box-shadow和drop-shadow滤镜：

```css
border: 10px dashed #beceeb; box-shadow: 5px 5px 10px black;
border: 10px dashed #beceeb; filter: drop-shadow(5px 5px 10px black);
```

结果：

![img](https://s2.loli.net/2022/05/06/LtalIVWiQsrhXe1.png)

`box-shadow`中间有透明部分也不会有阴影，只是一整个盒子有阴影。

而`drop-shadow`可以穿透透明部分，比较符合真实世界的投影。

添加投影效果

![image-20220506145029739](https://s2.loli.net/2022/05/06/GdADwm8jXot5sUl.png)

没有投影

![image-20220506145056283](https://s2.loli.net/2022/05/06/E1Osd9AhRGgYfXe.png)