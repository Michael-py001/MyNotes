## 20210915-scroll-view的正确使用方法

```html
<scroll-view scroll-x="true" class="scroll-x">
    <view class="img-wrap-one" v-for="(url,index) in imgList" :key="index" @click.stop="previewImg(index)">
        <image :src="url" mode="aspectFill"></image>
	</view>
</scroll-view>
```

## 横向滚动

### 方案一

设置`scroll-x="true"` ，

给`scroll-view`添加css属性：

​    `white-space: nowrap;` 强制让该行元素不能换行，只能一行显示；

给里面滑动的每个子元素添加css属性：

​	`display: inline-block;`  让子元素变成行内块元素，保持一行显示

​	

```css
.scroll-x {
    white-space: nowrap;
    width: 100%;
    height: 120rpx;
    .img-wrap-one {
        display: inline-block;
        width: 120rpx;
        height: 120rpx;
        margin-right: 18rpx;
        overflow: hidden;
        image {
            width: 100%;
            height: 100%;
        }
    }
}
```

### 方案二

```html
<scroll-view :scroll-x="true">
    <view class="list">
        <view class="item" v-for="(item, index) in list" :key="index" @click="itemClick(item)">
            xxx
        </view>
    </view>
</scroll-view>
```

设置`scroll-x="true"` ，

里面多包一层list，给list设置属性：

`display: flex;` 使子元素变成弹性元素，一行排列；

`width: max-content` ： width:max-content表示采用内部元素宽度值最大的那个元素的宽度作为最终容器的宽度。如果出现文本，则相当于文本不换行

```css
display: flex;
width: max-content;
```

子元素无需设置其他CSS。

## 纵向滑动

使用竖向滚动时，需要给 `<scroll-view>` 一个固定高度，通过 css 设置 height；

scroll-view是区域滚动，不会触发页面滚动，无法触发pages.json配置的下拉刷新、页面触底onReachBottomDistance、titleNView的transparent透明渐变。

