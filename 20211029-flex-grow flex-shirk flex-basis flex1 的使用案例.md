# 20211029-flex-grow flex-shirk flex-basis flex1 的使用案例



```html
<div id="content">
      <div class="box" style="background-color: red">A-box</div>
      <div class="box" style="background-color: lightblue">B-box</div>
      <div class="box" style="background-color: yellow">C-box</div>
      <div class="box1" style="background-color: brown">D-box1</div>
      <div class="box1" style="background-color: lightgreen">E-box1</div>
      <div class="box" style="background-color: brown">F-box</div>
    </div>
```



## flex-grow

grow：顾名思义 增大，flex-grow就是flex弹性盒子中的增长系数，数值越大，空间占的比例越大

```css
.box {
    flex-grow: 1;
    border: 3px solid rgba(0, 0, 0, 0.2);
}

.box1 {
    flex-grow: 3;
    border: 3px solid rgba(0, 0, 0, 0.2);
}
```

![image-20211120150248397](https://i.loli.net/2021/11/20/QabhJHmorejIqYc.png)

## flex-shirk

**`flex-shrink`** 属性指定了 flex 元素的收缩规则。**flex 元素仅在默认宽度之和大于容器的时候才会发生收缩**，其收缩的大小是依据 flex-shrink 的值。数值越大，就越缩小，与`flex-grow`是相反的。

注意：只有子元素的宽度大于父元素时，才会发生收缩。

```css
    #content {
        padding: 100px 30px;
        width: 300px;
        display: flex;
        border: 1px solid red;
      }
      div{
        width: 100px;
      }
      .box {
        flex-shrink: 8;
        border: 3px solid rgba(0, 0, 0, 0.2);
      }

      .box1 {
        flex-shrink: 1;
        border: 3px solid rgba(0, 0, 0, 0.2);
      }
```



![image-20211120171336011](https://i.loli.net/2021/11/20/Klqc8QyfCnY2orb.png)



## flex1

相当于width为剩余空间的宽度。
