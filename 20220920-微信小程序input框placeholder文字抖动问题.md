# 20220920-微信小程序input框placeholder文字抖动问题

input框文字抖动现象

![20220920](https://f.pz.al/pzal/2022/09/20/fa05f427f46fd.gif)

修复后的效果

![202209202](https://pic.imgdb.cn/item/63297d4816f2c2beb1c2533e.gif)

解决方案：

- 给placeholder添加样式，和Input框的设置一致

- 样式需要包含固定好高度，因为input自带高度
- 如果想要设置文字方向，最好使用lfex布局设置

```css
.input , .input-placeholder{
            height: 75rpx !important;
            display: flex;
            align-items: center;
            text-align: center;
            justify-content: center;
          }
```

