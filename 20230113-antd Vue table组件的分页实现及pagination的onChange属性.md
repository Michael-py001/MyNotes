# 20230113-antd Vue table组件的分页实现及pagination的onChange属性

## table组件分页如何实现？

> [Table组件](https://2x.antdv.com/components/table-cn/#API)

只需要给table组件传入`pagination`属性，内部会自动处理页码的显示:

![image-20230113105513468](https://s2.loli.net/2023/01/13/7ypSuHj8rhw6x15.png)

```html
<a-table 
      :dataSource="dataSource" 
      :columns="columns" 
      :pagination="pagination"
      @change="handleChange"
      bordered
    >
```

```js
 let pagination = reactive({
    total: totalItems,
    current: currentPage,
    pageSize: perPage,
  })
```

## 如何实现切换页码？

### 1、使用table的`change`事件

在table中，可以使用table的`change`事件，在传入了`pagination`属性后，点击切换页码就会触发这个事件，你可以在回调里面进行更新`pagination`的`current`属性，就会更新页码。

![image-20230113105803979](https://s2.loli.net/2023/01/13/pN8Df7cedOKsRaj.png)

```js
const queries = reactive({
    current: 1,
    size: 10,
});
// 切换页码
const handleCurrentChange = ({ current, pageSize }) => {
    queries.current = current;
    queries.size = pageSize;
};
```

## 2、在pagination中添加onChange事件

```js
 // 页码数据
  let pagination = reactive({
    total: totalItems,
    current: currentPage,
    pageSize: perPage,
    onChange: (num,size) => {
      console.log(num,size);
    }
  })
```

​		在antd Vue的pagination文档中，是没有写到有`onChange`事件的，只写了`change`事件。而现在我们是在table组件中使用，需要在table中传入pagination，而不是使用pagination组件，如果你是在模板中使用组件的形式写pagination，就可以直接写`@change`就可以触发。但是要在table中传入pagination，必须要遵循JSX语法，就是要改成onChange的写法才是正确可以触发的。

![image-20230113110534367](https://s2.loli.net/2023/01/13/5SVbC1MIiekwRug.png)

在gihub的仓库中已经有不止一个人发现了这个问题，并提交了pullRequest，但是被官方的人拒绝了，详情可以看以下的两个链接：

![image-20230113111416797](https://s2.loli.net/2023/01/13/xL7IrHYusE9MgSk.png)

![image-20230113111402937](https://s2.loli.net/2023/01/13/KNa7LOdysuvFo14.png)

> [fix: function name in pagination document is wrong by Plane-walker · Pull Request #1053 · vueComponent/ant-design-vue (github.com)](https://github.com/vueComponent/ant-design-vue/pull/1053)
>
> [doc: correct invaild name of api by tzhiy · Pull Request #5094 · vueComponent/ant-design-vue (github.com)](https://github.com/vueComponent/ant-design-vue/pull/5094)

## showTotal的使用

```js
let pagination={
    showQuickJumper: true,
    current: queries.current,
    total: tableTotal,
    showTotal: total => `共 ${tableTotal} 条`,
    pageSize: queries.size,
}
```

![image-20230113110435920](https://s2.loli.net/2023/01/13/J4XKhRkCxQV1ODI.png)