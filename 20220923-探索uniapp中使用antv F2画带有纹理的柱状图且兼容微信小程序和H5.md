# 20220923-探索uniapp中使用antv F2画带有纹理的柱状图且兼容微信小程序和H5

这是官方提供的demo地址: [纹理柱状图 | F2 (antv.vision)](https://f2-v3.antv.vision/zh/examples/column/column#pattern)，效果长这样：

![image-20220923144844943](https://f.pz.al/pzal/2022/09/23/af9aeda6c688b.png)

在我封装的uniapp组件库里，我要兼容小程序和H5的显示，以下是我遇到的问题并一步步解决的过程。

## 首次报错

关键代码如下：

```js
// 创建纹理对象
// 获取 canvas 上下文对像
const ctx = chart.get('canvas').get('context');
const img = new Image();
img.src = 'https://gw.alipayobjects.com/zos/rmsportal/cNOctfQVgZmwaXeBITuD.jpg';
img.onload = function() {
  const pattern = ctx.createPattern(img, 'repeat');
  chart.interval().position('year*sales').color(pattern);
  chart.render();
};
```


![image](https://user-images.githubusercontent.com/60598432/191717647-04732638-a058-4b09-82ea-d499777fbbf2.png)](https://user-images.githubusercontent.com/60598432/191717647-04732638-a058-4b09-82ea-d499777fbbf2.png)
[![image](https://user-images.githubusercontent.com/60598432/191718571-9ba0b1aa-951d-4f1e-af33-31f36d0aba24.png)](https://user-images.githubusercontent.com/60598432/191718571-9ba0b1aa-951d-4f1e-af33-31f36d0aba24.png)

[![image](https://user-images.githubusercontent.com/60598432/191717995-897feb40-146d-4cc0-b8a2-f34ac453ab01.png)](https://user-images.githubusercontent.com/60598432/191717995-897feb40-146d-4cc0-b8a2-f34ac453ab01.png)



当我复制上面的代码到Uniapp中运行时，在H5端发生以下报错，很明显，以下的图显示 new Image()一个节点被uniapp解析当成filePath解析导致报错，我以为这个是uniapp的bug，随即我就去github提了个issue，很快，他们的技术人员回复了我：

![Snipaste_2022-09-22_18-21-16](https://f.pz.al/pzal/2022/09/23/a5aed7555b736.jpeg)

[CanvasContext.createPattern文档](https://uniapp.dcloud.net.cn/api/canvas/CanvasContext.html#canvascontext-createpattern)

微信小程序的文档也是同样的说明

![image-20220923165508232](https://raw.githubusercontent.com/Michael-py001/imgUpload/matster/img/202209231655872.png)

## H5的写法

原来是createPattern的只能是包内的资源路径和图片的临时路径。

所以H5的兼容写法如下

```js
const pattern = ctx.createPattern('/static/Chart/test.jpg', 'repeat');
chart.interval().position('year*sales').color(pattern);
chart.render();
```

运行效果：

![image-20220923153948286](https://f.pz.al/pzal/2022/09/23/09c924135038e.png)

## 微信小程序的写法

如果用上面的H5写法编译到小程序里，又会报错不兼容：

Failed to execute 'createPattern' on 'CanvasRenderingContext2D'

![image-20220923152444785](https://f.pz.al/pzal/2022/09/23/b206ebb31a89c.png)

我个人觉得是因为，我使用的是微信小程序的新版2D canvas，他获取到的上下文是一个CanvasRenderingContext2D类型的对象。而浏览器里的canvas对象是：CanvasContext类型，所以这两者不能通用。

![image-20220923152701025](https://f.pz.al/pzal/2022/09/23/2578f225264fb.png)

于是我想到了用canvas加载图片的方法，试了一下，果然可以：

```js
const canvasImg = canvas.createImage();
canvasImg.onload = (e) => {
    // #ifdef MP-WEIXIN
    const pattern = ctx.createPattern(canvasImg, 'repeat');
    // #endif
    chart.interval().position('year*sales').color(pattern);
    chart.render();
}
canvasImg.onerror = (e) => {
    console.error('err:', e)
}
canvasImg.src = '/static/Chart/test.jpg'
```

![image-20220923164304999](https://f.pz.al/pzal/2022/09/23/826a25b201da4.png)

但是这里有个疑问，我使用网络图片资源路径在小程序和H5也能运行，这跟文档写的本地路径才能用不一致啊，具体什么原因久不清楚了。

```js
//h5
const pattern = ctx.createPattern('https://gw.alipayobjects.com/zos/rmsportal/cNOctfQVgZmwaXeBITuD.jpg', 'repeat');
```

```js
//mp
canvasImg.src = 'https://gw.alipayobjects.com/zos/rmsportal/cNOctfQVgZmwaXeBITuD.jpg'
```

![image-20220923170545771](https://raw.githubusercontent.com/Michael-py001/imgUpload/matster/img/202209231705445.png)
