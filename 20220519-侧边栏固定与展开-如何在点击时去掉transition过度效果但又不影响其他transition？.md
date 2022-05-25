# 20220519-侧边栏固定与展开-如何在点击时去掉transition过度效果但又不影响其他transition？

如下图的效果，固定侧边栏是使用`position:fixed`属性控制，展开效果是使用`margin-left`，都使用`transition`添加动画过渡效果。

但是可以看到刚开始未处理时，点击固定图标，设置`fixed`时，也会触发过渡动画，这样会让页面发生抖动，很不协调。

处理前的效果

![move222](https://s2.loli.net/2022/05/19/TrBhml6vWqRkZa9.gif)

处理后的效果：

![move1](https://s2.loli.net/2022/05/19/gSwjULeOcuEhGxC.gif)



## 解决过程

我们可以在点击时，把这个div的`transition`去掉，等之后再设置回去，这样瞬间发生用户是不会有任何察觉的。

同时，设置回去时需要用宏任务，比如用`setTimeout`，要等页面的回流重绘结束后再设置，不然浏览器会把两条设置逻辑合并成一次设置，这是浏览器自身的优化逻辑。

```js
handleFixed() {
    this.$refs.childAside.style.transition = 'none'
    this.isFixed=!this.isFixed
    setTimeout(()=>{
        this.$refs.childAside.style.transition = '0.3s'
    })
},
```































