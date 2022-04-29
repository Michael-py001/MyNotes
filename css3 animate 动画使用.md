

# css3 animate 动画使用

```scss
 .showAnimate {
        display: flex;
        opacity: 1;
        -webkit-transform-style: flat;//规定如何在 3D 空间中呈现被嵌套的元素。flat preserve-3d
     //flat	子元素将不保留其 3D 位置。
     //preserve-3d	子元素将保留其 3D 位置。
        -webkit-transform-origin: center;//中心点
        -webkit-animation: show 1s linear 0s 1 normal forwards;
     								//名字 动画时间 动画效果 延迟秒数 执行几次 执行后保持最后一帧
        backface-visibility: hidden;//隐藏被旋转的 div 元素的背面：
        -webkit-animation-timing-function:cubic-bezier(0.25,0.1,0.25,1);//规定动画的速度曲线
        @keyframes show {
           0% {
             transform: scale(0.5);
           }
           50% {
             transform: scale(1.2);
           }
           100% {
             transform: scale(1);
           }
         }
      }
```

