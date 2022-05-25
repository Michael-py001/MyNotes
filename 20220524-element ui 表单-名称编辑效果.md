host20220524-element ui 表单-名称编辑效果

实现的效果：

点击编辑按钮，标题名称从普通文本激活成`input`输入框，并自动填入该名称，失焦后或者再次点击按钮，保存输入的名称

![image-20220524184247731](https://s2.loli.net/2022/05/24/UbVgldCJxuWPGyQ.png)

![image-20220524184301664](https://s2.loli.net/2022/05/24/Keg2fcl68Eb4OAy.png)



## 实现的代码

```html
<el-table-column property="name" label="标题名称" width="248">
    <template #default="scope">
        <div class="name-wrap">
            <el-input change="input" v-model="scope.row.name" size="medium" placeholder="请输入标题名称"
                      v-show="scope.row.edit" @keyup.enter.native="titleWordRename" />
            <el-tooltip class="item" effect="light" :content="scope.row.name" placement="bottom-start">
                <div class="name u-line-1" v-show="!scope.row.edit">{{scope.row.name}}</div>
            </el-tooltip>

            <div class="edit" @click="onEditName(scope.row)">
                <img class="icon" :src="require('assets/images/public/edit.png')">
            </div>
        </div>
    </template>
</el-table-column>
```

自定义`edit`属性，用来判断激活状态

```js
onEditName(i) {
    console.log(i);
    if (i.edit) {
        this.titleWordRename()
    } else {
        this.activeItemId = i.id
        i.edit = !i.edit
        i.tempName = i.name
    }
}
```

