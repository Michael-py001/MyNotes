# 20220509-box-sizing：border-box与content-box的区别

## content-box

首先看`content-box`，根据以下的实际案例可以得出结论：

`content-box`会把盒子的各部分：width , padding , border ,  margin 独立计算

- 设置width/height 是把盒子本身看作主体内容，设置盒子自身的宽高
- 设置padding是看作给盒子附加的内容，会设置额外的宽高
- 设置border是在padding外添加额外边距
- 设置margin是在width/height / padding 之外设置距离其他元素的间距

以上各部分都相互独立，比较符合现实中的计算规律。 

主要的css:

```css
padding:10px;
width: 100px;
height: 100px;
border: 1px solid rgb(185, 18, 18);
margin: 0 auto;
```

![image-20220509113725788](https://s2.loli.net/2022/05/09/P6wkLhxXebTA9M4.png)

![image-20220509113918898](https://s2.loli.net/2022/05/09/mDv7B2euocJWblH.png)

总结width计算规则= `width` = `content-width` ，也就是说你设置了width是多少，盒子的内容高度就是多少

## border-box

`border-box`被称作IE盒模型/怪异盒模型，主要的特点就是设置该属性的盒子，`width`的计算规则改变成：`width` = `content-width` + `padding` + `border` ，也就是说如果你设置了width，该盒子的宽度会包含border , padding 在内一起计算，最后`content-width`会自动计算成剩余的宽度。

![image-20220509114817554](https://s2.loli.net/2022/05/09/p1jPU2875EGyulv.png)

![image-20220509114840290](https://s2.loli.net/2022/05/09/Bt7ufaCH6JFAen4.png)