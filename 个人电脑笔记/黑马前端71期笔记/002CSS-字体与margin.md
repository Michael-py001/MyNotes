## CSS的一些属性

### 	1. font-style

```css
.p1{
	font-style: normal;/*正常显示*/
}
.p2{
	font-style: italic; /*如果英文字体有斜体，则显示斜体，若没有，则替换成有斜体的英文字体*/
}
.p3{
	font-style: oblique;/*字体倾斜，和字体斜体没有关系*/
}
```

![image-20200719190450896](https://i.loli.net/2020/07/19/gH1vtRsbYeNEFfD.png)

### 2. text-decoration

```css
a{
    text-decoration: none;/*去除默认样式*/
}
.p2{
	text-decoration: underline; /*下划线*/
}
.p3{
	text-decoration: line-through;/*删除线*/
}
.p4{
	text-decoration: overline;/*上划线*/
}
```

![image-20200719191018812](https://i.loli.net/2020/07/19/gc4uKnlzUiWPJSa.png)

### 3. font 多个字体属性

```css
p{			
    height: 50px;
    color: #252525;
    font: bold italic 26px/50px "Arial";/*加粗 斜体 字号26px 行高50px 字体*/			
}
```

### 4.margin塌陷

1. **上下盒子margin塌陷**

   > 上下盒子发生margin塌陷，margin小的盒子塌陷在大的盒子 不是简单的叠加

   ```html
   <div class="box1">1</div>
   <div class="box2">2</div>
   ```

   ```css
   .box1 {
   width: 200px;
   height: 200px;
   background: yellowgreen;
   margin-bottom: 100px;   /*margin小的box*/
   }
   
   .box2 {
   width: 200px;
   height: 200px;
   background: skyblue;
   margin-top: 150px;  /*margin大的box，两者不会叠加，而是以大的为准*/
   }
   
   ```

   <img src="https://i.loli.net/2020/07/20/Ihw2kZsv51E8c6S.png" alt="image-20200720183743038" style="zoom:50%;" />

2. **父子盒子margin塌陷**

   ```html
   <div class="box1">
   	<div class="box2"></div>
   </div>
   ```

   ```css
   .box1{
       width: 200px;
       height: 140px;
       background: yellowgreen;
       margin-top: 100px;  /*以大的为准*/
   }
   .box2{
       width: 100px;
       height: 100px;
       background: skyblue;
       margin-top: 60px; 
   }
   ```

   <img src="https://i.loli.net/2020/07/20/JUjCl7SQn3GdP49.png" alt="image-20200720184048650" style="zoom: 80%;" />

   如果想要的效果是子盒子距离父盒子顶部60px，此时可以给父盒子添加padding值。

   ```css
   .box1 {
       width: 200px;
       height: 140px;
       padding-top: 60px;
       background: yellowgreen;
   }
   
   .box2 {
       width: 100px;
       height: 100px;
       background: skyblue;
   }
   ```

   <img src="https://i.loli.net/2020/07/20/8eT9YUlIWqsuKRp.png" alt="image-20200720184931199" style="zoom:80%;" />

   所以父子盒子中，善用父亲的padding，少用儿子的margin