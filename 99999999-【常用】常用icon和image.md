图标

```html
<u-icon name="arrow-right" size="28" color="#c4c4c4"></u-icon>  //右箭头
<u-icon name="map" size="35"  color="#888888"></u-icon> //地图
<u-icon name="arrow-up" size="35"></u-icon> //上箭头
<u-icon name="arrow-down" size="35"></u-icon> //x
<u-icon name="search" size="35"></u-icon> //搜索
<u-icon name="close" size="35"></u-icon>  //关闭×
```



图片

```html
<u-image width="88rpx" height="88rpx" border-radius="50%" :src="info.avatar" ></u-image>
    
<u-image width="160rpx" height="160rpx" border-radius="50%" src="https://codefun-proj-user-res-1256085488.cos.ap-guangzhou.myqcloud.com/617a2ca7e4f1450011362b37/61db95b5a92fb70011747062/16417809285654041162.png" >
        <template #loading>
         <u-loading ></u-loading>
      </template>
      </u-image>
```

选中框

```html
 <u-checkbox  v-model="item.checked" @change="checkboxChange" shape="circle" size="40" active-color="#4CBCA5">{{item.name}}</u-checkbox>
```

![评分 (2)](C:/Users/PM/Downloads/%E8%AF%84%E5%88%86%20(2).png)
