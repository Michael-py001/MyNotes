## 底部按钮高度

```html
	<view class="cart-panel" :style="'bottom:'+windowBottom+'px;'"></view>
```

```css
.cart-panel {
		z-index: 99;
		background-color: #fff;
		width: 100%;
		height: 100upx;
		position: fixed;
		left: 0upx;
		display: flex;
		justify-content: space-between;
		align-items: center;
	}
```

可以在WEB端计算底部距离

```js
//计算底部距离，兼容web页面
let systemInfo = uni.getSystemInfoSync()
this.windowBottom = systemInfo.windowBottom ? systemInfo.windowBottom : 0
```

![image-20210611174518853](https://i.loli.net/2021/06/11/N9ceEYmi8szpyXU.png)