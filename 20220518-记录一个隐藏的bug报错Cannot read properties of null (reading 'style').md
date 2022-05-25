# 20220518-记录一个隐藏的bug报错:Cannot read properties of null (reading 'style')

在项目中，切换菜单的路由时，出现了一个无法定位的报错

![image-20220518165447813](https://s2.loli.net/2022/05/18/uKbADgLywHY4miv.png)

```js
this.$router.replace({
    path: url
})
```

切换路由是切换成功了，但是页面没有出来，也就是说这个报错导致组无法正常渲染。

## 解决过程

### 1. 搜索关键字

既然是有`style`的提示，那就在各个页面搜索`style`的关键字，在一处地方找到了似乎是报错的地方：

```html
	<el-table-column property="count" label="所属账号" width="180">
        <template #default="scope">
            <el-tooltip class="item" effect="light" 
                        :content="scope.row.advertiser_ids.map(i=>i.name) + scope.row.advertiser_ids.map((i,index)=> {index == scope.row.advertiser_ids.length - 1 ? '' : '，' } )" 
                        placement="bottom">
                <div class="advertiser-list">
                    <div class="u-line-1" :style="{display: index > 0 ? 'none' : ''}" v-for="(i,index) in scope.row.advertiser_ids" :key="index">{{i.name}}</div>
                </div>
            </el-tooltip>
        </template>
</el-table-column>
```



我把下面这一段注释掉后：

```html
<div class="u-line-1" :style="{display: index > 0 ? 'none' : ''}" v-for="(i,index) in scope.row.advertiser_ids" :key="index">{{i.name}}</div>
```

发现还是有报错的存在。可能报错的地方不在这？继续寻找。。。

### 2. 使用注释大法

既然报错给出的信息找不到出错的地方，那就使用注释大法，把模板一处处注释，不断缩小范围。

同时看到了这篇博客，受到启发：

> [Solve - Cannot read property 'style' of Null in JavaScript | bobbyhadz](https://bobbyhadz.com/blog/javascript-cannot-read-property-style-of-null#:~:text=The "Cannot read property 'style' of null" error,tag before the DOM elements have been declared)

最终找到了问题出现的地方，原来问题还是出现在上面的那段html，但是刚刚我只注释了其中包含`style`的一段。这次我把一整段注释后，发现报错没有出现了。根据我的经验，根据我的经验，模板中含有复杂的函数逻辑时，极容易报错。

我果断把这一段代码抽离出来：

`:content=scope.row.advertiser_ids.map(i=>i.name) + scope.row.advertiser_ids.map((i,index)=> {index == scope.row.advertiser_ids.length - 1 ? '' : '，' } )`

改成单独的函数处理：

```js
handleTooltip(scope) {
    let name = scope.row.advertiser_ids.map(i=>i.name) + scope.row.advertiser_ids.map((i,index)=> {index == scope.row.advertiser_ids.length - 1 ? '' : '，' } )
    return name || ''
},
```

原来的模板改成：

```html
<el-tooltip class="item" effect="light" 
            :content="handleTooltip(scope)" 
            placement="bottom">
    <div class="advertiser-list">
        <div class="u-line-1" :style="{display: index > 0 ? 'none' : ''}" v-for="(i,index) in scope.row.advertiser_ids" :key="index">{{i.name}}</div>
    </div>
</el-tooltip>
```

但是，快速切换路由的时候，还是会出现报错。。。

### 3. 属性传值更改为模板传值

经过查看文档，发现`el-tooltip`是可以通过`slot`传模板显示的，使用模板比使用`content`属性传值报错的几率小很多，基本很稳定，因为是直接显示html，不经过js处理。

```html
<el-tooltip class="item" effect="light" 
            placement="bottom"
            >
    <template #content>
        {{handleTooltip(scope)}}
    </template>
    <div class="advertiser-list" >
        <div class="u-line-1" :style="{display: index > 0 ? 'none' : ''}" v-for="(i,index) in scope.row.advertiser_ids || []" :key="index">{{i.name}}</div>
    </div>
</el-tooltip>
```

使用`#content`插槽显示后，报错终于消失了。