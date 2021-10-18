## 20210915-CSS3中的fill-available、max-content、min-content、fit-content

一般地，有两种自适应：撑满空闲空间与收缩到内容尺寸。CSS3将这两种情况分别定义为'fill-availabel'和'fit-content'。除此之外 ，还新增了更细粒度的'min-content'和'max-content'。这四个关键字可用于设置宽高属性。本文将详细介绍CSS3中的这四个自适应关键字

　　[注意]IE浏览器不支持，webkit内核浏览器需添加-webkit-前缀

## **fill-available**

`width:fill-available`表示撑满可用空间

比如，一个`div`标签是块级元素，默认就是自动填满剩余空间

而`fill-available`的价值在于，它可以让元素100%自动填充的特性应该用在其他元素上，不仅仅是block元素

```html
<style>
    span{
      background-color: pink;
      display:inline-block;
      width:-webkit-fill-available;
    }
</style>
<span>这是一段文字...</span>
```

使用对比，添加`width:-webkit-fill-available`后，内联元素span的宽度填满了可用宽度

![image-20210915181445456](https://i.loli.net/2021/09/15/VGprf9DtYBsw3A1.png)![image-20210915181513918](https://i.loli.net/2021/09/15/mOnwtUJ7fSquY2P.png)

高度也有此特性

![image-20210915181934970](https://i.loli.net/2021/09/15/UyLfgOmHiEnkTDP.png)

![image-20210915182327325](https://i.loli.net/2021/09/15/gha8lCO4psrTVzG.png)

### 【等高布局】

可以利用此特性实现登高布局

```html
<style>
    .inner{
        width:100px;
        height:-webkit-fill-available;
        margin:0 10px;
        display: inline-block;
        vertical-align: middle;
        background-color: pink;
    }
</style>
<div style="height: 100px;">
    <div class="inner">HTML</div>
    <div class="inner">CSS</div>
    <div class="inner">JS<br>jQyery<br>Vue</div>
</div>
```

![image-20210915182846306](https://i.loli.net/2021/09/15/NqIMcwuVODilL2s.png)

## **fit-content**

width:fit-content表示将元素宽度收缩为内容宽度，同时保持原本的block特性 ，与**fill-available**刚好相反

```html
<style>
    div{
        background-color: pink;
        width:-webkit-fit-content;
    }
</style>
<div>这是一段文字</div>
```

没有使用前，div是不满宽度的，使用后，宽度缩小为内容的宽度

![image-20210915183043476](https://i.loli.net/2021/09/15/UNOalED6b4iRqp9.png)

![image-20210915183058081](https://i.loli.net/2021/09/15/CXKBtwi1QrSN9Ek.png)

### 【水平居中】

`　width:fit-content`可以实现元素收缩效果的同时，保持原本的block水平状态，于是，就可以直接使用`margin:auto`实现元素向内自适应同时的居中效果了



![image-20210915183438233](https://i.loli.net/2021/09/15/scL4pBShg2kxOAy.png)

高度也有此特性



## **min-content**

width:min-content表示采用内部元素最小宽度值最大的那个元素的宽度作为最终容器的宽度

首先，要明白这里的“最小宽度值”是什么意思。替换元素，例如图片的最小宽度值就是图片呈现的宽度，对于文本元素，如果全部是中文，则最小宽度值就是一个中文的宽度值；如果包含英文，因为默认英文单词不换行，所以，最小宽度可能就是里面最长的英文单词的宽度

```html
<style>
    .outer{
        width:-webkit-min-content;
    }
</style>
<div class="outer">
    <div style="height:10px;width:100px;background:lightgreen"></div>
    <div style="background-color: pink;">这是一段文字</div>
</div>
```

![image-20210915184240273](https://i.loli.net/2021/09/15/K4t7WriOANYdzZX.png)

![image-20210915184257228](https://i.loli.net/2021/09/15/Olu8mGISYKfZJ4o.png)

## **max-content**

width:max-content表示采用内部元素宽度值最大的那个元素的宽度作为最终容器的宽度。如果出现文本，则相当于文本不换行

```html
<style>
    .outer{
        width:-webkit-max-content;
        border:1px solid black;
    }
</style>
<div class="outer">
    <div style="height:10px;width:100px;background:lightgreen"></div>
    <div style="background-color: pink;">这是一行文字。。这是一行文字。。这是一行文字。。这是一行文字。。这是一行文字。。这是一行文字。。这是一行文字。。</div>
</div>
```

![image-20210916093527657](https://i.loli.net/2021/09/16/SOPKqoI2FvtRV87.png)

![image-20210915184503576](https://i.loli.net/2021/09/15/hqiOvStAzyarsdj.png)

