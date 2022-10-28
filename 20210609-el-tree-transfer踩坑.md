# el-tree-transfer踩坑

> 官方文档:https://github.com/hql7/tree-transfer

老项目中用到了这个组件，发现有个Bug，有时候左边的数据添加到右边，左边的消失了，但是没有添加到右边。

![image-20210609102311513](https://i.loli.net/2021/06/09/6PzwldbEi3KXpeQ.png)

最后发现是数据格式的问题。

官方提供的数据格式：

```json
 fromData:[
     {
         id: "1",
         pid: 0,
         label: "一级 1",
         children: [
             {
                 id: "1-1",
                 pid: "1",
                 label: "二级 1-1",
                 disabled: true,
                 children: []
             },
             {
                 id: "1-2",
                 pid: "1",
                 label: "二级 1-2",
                 children: [
                     {
                         id: "1-2-1",
                         pid: "1-2",
                         children: [],
                         label: "二级 1-2-1"
                     },
                     {
                         id: "1-2-2",
                         pid: "1-2",
                         children: [],
                         label: "二级 1-2-2"
                     }
                 ]
             }
         ]
     },
 ],
```

接口返回的数据：

![image-20210609102946295](https://i.loli.net/2021/06/09/uy1OcaroEAHCVQ5.png)

![image-20210609103057663](https://i.loli.net/2021/06/09/bGB89yMkmgRVUS3.png)

前端从接口获取数据后自己构建数据格式：

```js
 async getServiceIds () {
        let { data } = await this.$Get('Company.getServiceIds')
        this.currentOptions = data.ids
        data.items.forEach((item,index) => {
          let childList = item.child.map((childItem, childIndex) => {
            return {
              pid: item.id,
              // id: `${item.id}-${childIndex}`,//之前的错误使用
              id: `${item.id}-${childItem.id}`,//这个id必须是唯一的,而且格式需要是：pid-id，没有pid会不能识别是哪个父层
              childId: childItem.id,//自定义的属性
              label: childItem.name
            }
          })
          this.existInfo.push({
            id: item.id,  //最外层的id和子数据childList里的每一项pid必须一致
            label: item.name,
            pid: 0, //最外层的pid必须是0，因为已经是最外层了，没有父数据
            children: childList
          })
        })
        console.log("this.existInfo：",this.existInfo)
      },
      // 获取全部服务
      async getAllService () {
        let { data } = await this.$Get('Company.getAllService')
        data.items.forEach((item,index) => {
          let childList = item.child.map((childItem, childIndex) => {
            return {
              pid: item.id,
              // id: `${item.id}-${childIndex}`,//之前的错误使用
              id: `${item.id}-${childItem.id}`,//必须使用唯一id
              childId: childItem.id,
              label: childItem.name
            }
          })
          this.serveInfo.push({
            id: item.id,
            label: item.name,
            pid: 0,
            children: childList
          })
        })
        console.log("this.serveInfo：",this.serveInfo)
      },
```

