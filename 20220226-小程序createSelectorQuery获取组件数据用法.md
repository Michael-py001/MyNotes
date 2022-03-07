# 20220226-小程序createSelectorQuery获取组件数据用法

## fields

```js
 const query = wx.createSelectorQuery().in(this);
    let mainNode = query.select('.container')
    mainNode.fields({
        id: true,
        dataset: true,
        rect: true,
        size: true,
        scrollOffset: true,
        context: true
    }, res => {
        ...
    }).exec()
```

## boundingClientRect

```js
 async getHeight () {
        return new Promise(resolve=>{
          const query = uni.createSelectorQuery().in(this)
          query.select('.News').boundingClientRect((res)=>{
            console.log("res:",res)
            let {height} = res
            this.$emit('getHeight',height)
            resolve(true)
          }).exec()
        })

      }
```

