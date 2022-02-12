# 20220209-伪类实现虚线替代border dashed

最终效果

![image-20220210110334920](https://s2.loli.net/2022/02/10/RzlHECuTQOLxKJ8.png)

![image-20220210111406374](https://s2.loli.net/2022/02/10/N9KvQ4lUOz57MCX.png)

> 使用border: 1rpx dashed #e5e5e5;不能控制虚线的长度 间隙，最终使用伪类+linear-gradient实现虚线

```


```

代码

```css
.right {//右边立即领取的区域
          text-align: center;
          width: 84rpx;
          // border-left: 1rpx dashed #e5e5e5;;
          color: #839C80;
          font-size: 24rpx;
          padding: 10rpx 30rpx;
          box-sizing: border-box;
          display: flex;
          justify-content: center;
          align-items: center;
          flex-wrap: wrap;
          position: relative;
          box-sizing: border-box;
          &::before {
            content: ' ';
            height: 100%;
            width: 200rpx;
            position: absolute;
            left: 0;
            top: 0;
              background-image: linear-gradient(
                to bottom,
                #E5E5E5 0%,
                #E5E5E5 50%,
                transparent 50%
              );
              background-size: 1px 13px;
              background-repeat: repeat-y;
            }
          .text {
            line-height: 1.6;
            word-wrap: break-word;
            font-size: 24rpx;
          }
				}
```

核心代码

```css
&::before {
    content: ' ';
    height: 100%;
    width: 200rpx;
    position: absolute;
    left: 0;
    top: 0;
    background-image: linear-gradient(
        to bottom,
        #E5E5E5 0%,
        #E5E5E5 50%,
        transparent 50%
    );
    background-size: 1px 13px;//x y代表width height y越大 虚线越长
    background-repeat: repeat-y; //竖向排列
}
```

