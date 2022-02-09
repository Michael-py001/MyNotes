## scroll-view跟随滚动

> https://uniapp.dcloud.io/component/scroll-view?id=scroll-view

# 20211210-scroll-view跟随滑动

## scroll-into-view属性

值应为某子元素id（id不能以数字开头）。设置哪个方向可滚动，则在哪个方向滚动到该元素

给每个元素设置id 点击改变k，即可滚动到指定的item

```js
:id="'item-'+k"
```



## scroll-with-animation 属性

在设置滚动条位置时使用动画过渡

```html
<scroll-view class="scroll-view u-skeleton-fillet" :scroll-x="true" :scroll-into-view="'item-'+active"
				scroll-with-animation>
    <view :id="'item-'+k" class="item" :class="{'work':i.work,'active':active===k}" v-for="(i,k) in list"
          :key="k" @click="handleItem(i,k)">
        <view class="week">
            {{i.week_str}}
        </view>
        <view class="info">
            <view class="day">
                {{i.day}}
            </view>
            <view class="status" :class="{'active':!i.isRest}">
                {{i.isRest?'休息':'营业'}}
            </view>
        </view>
    </view>
</scroll-view>
```