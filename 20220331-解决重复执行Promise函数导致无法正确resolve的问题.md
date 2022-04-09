# 20220331-解决重复执行Promise函数导致无法正确resolve的问题

![move46](https://s2.loli.net/2022/03/31/2MmakEBxIg7OVZy.gif)

子组件中的方法：

```js
 start() {
      return new Promise((resolve,reject)=>{
        
        this.show = true
        if(this.timer) {
          clearInterval(this.timer)
        }
         this.timer = setInterval(()=>{
          console.log
          if(this.percent<90) {
            this.percent ++
          }
          else if(!this.finish&&this.percent==90) {
              // 停止增加
          }
          else if(this.finish&&this.percent>=90&&this.percent<100) {
            this.percent ++
          }
          else if(this.percent >= 100) {
            clearInterval(this.timer)
            this.timer = null
            this.$ShowToast('保存成功')
            this.show = false
            
            setTimeout(()=>{
              this.percent = 0
              this.resolveList = []
            },500)
          }
        },this.speed)
      })
     
    }
```

子组件中的watch监听父组件传递过来的接口是否处理完成的信号，这里会再次执行`this.start()`

```js
  watch:{
    finish(value) {
      if(value) {
        this.speed = 40
        clearInterval(this.timer)
        this.start()
      }
    }
  },
```

父组件中的async await等待结果

```js
let result = await this.$refs.Progress.start()
```

## 存在的问题及出现的原因

当子组件接收到`finish:true`时，会重复执行`this.start()`，return的函数已经不是第一次执行的函数，`Promise`也不是第一次`Promise` 了，导致`Promise`里面的`resolve`无法执行，所以父组件的`await`是永远也等不到`resolve`的了。

## 解决方案

由于是无法执行第一次的`resolve`，所以思路就是把这个`resolve`保存起来，等最后再拿出来执行，这样就不会有丢失的问题

优化后的代码：

```js
start() {
      return new Promise((resolve,reject)=>{
        this.resolveList.push(resolve)
        this.show = true
        if(this.timer) {
          clearInterval(this.timer)
        }
         this.timer = setInterval(()=>{
          console.log
          if(this.percent<90) {
            this.percent ++
          }
          else if(!this.finish&&this.percent==90) {
              // 停止增加
          }
          else if(this.finish&&this.percent>=90&&this.percent<100) {
            this.percent ++
          }
          else if(this.percent >= 100) {
            clearInterval(this.timer)
            this.timer = null
            this.$ShowToast('保存成功')
            this.show = false
            this.resolveList.forEach(i=>i(true)) //通知所有Promise状态改变
            setTimeout(()=>{
              this.percent = 0
              this.resolveList = []
            },500)
          }
        },this.speed)
      })
     
    }
```

