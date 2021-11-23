## -webkit-overflow-scrolling: touch

​		属性控制元素在移动设备上是否使用滚动回弹效果.  

> auto: 使用普通滚动, 当手指从触摸屏上移开，滚动会立即停止。
> touch: 使用具有回弹效果的滚动,当手指从触摸屏上移开，内容会继续保持一段时间的滚动效果。继续滚动的速度和持续的时间和滚动手势的强烈程度成正比。同时也会创建一个新的堆栈上下文。

​		在移动端上，在你用overflow-y:scorll属性的时候，你会发现滚动的效果很很生硬，很慢，这时候可以使用-webkit-overflow-scrolling:touch这个属性，让滚动条产生滚动回弹的效果，就像ios原生的滚动条一样流畅。 

### 问题

- 在safari上，使用了-webkit-overflow-scrolling:touch之后，页面偶尔会卡住不动。(中招)

- 在safari上，点击其他区域，再在滚动区域滑动，滚动条无法滚动的bug。
- 通过动态添加内容撑开容器，结果根本不能滑动的bug。(中招)
- 滚动中 scrollTop 属性不会变化
- 手势可穿过其他元素触发元素滚动
- 滚动时暂停其他 transition
  

解决方案：
方案一

```html
<div id="app" style="-webkit-overflow-scrolling: touch;">
    <div style="min-height:101%"></div>
</div>
```

 

方案二

```html
<div id="app" style="-webkit-overflow-scrolling: touch;">
    <div style="height:calc(100%+1px)"></div>
</div>
```

方法就是在webkit-overflow-scrolling:touch属性的下一层子元素上，将height加1%或1px。从而主动触发scrollbar。
