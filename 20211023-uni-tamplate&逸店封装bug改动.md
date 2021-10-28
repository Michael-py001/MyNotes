## 20211023-uni-tamplate&逸店封装bug改动

### 1. 【逸店】请求code错误时的回调

request.js文件

错误：原本是写成了不在返回码内 resolve( ) 而不是reject( )

![image-20211023173803803](https://i.loli.net/2021/10/23/ikXqRnAy85Tw6Dg.png)

## 2. 请求参数ignoreToastCode

```js
this.$Request("Sharing.SharingGoods", {
        goods_id: this.goodsId
    },{
        // codeHandle:{
        //   0:(res)=>{
        //     console.log("-------------:",res)
        //     this.$ShowToast("商品已下架").then(()=>{
        //       this.$nav('back')
        //     })
        //   }
        // },
        ignoreToastCode:[0],
        // toastHandle:true
    }
                 ).then(()=>())
```

文件：Request/lib.js -->CheckIgnoreToastCode

```js
原本 return !(_ignoreToastCode.indexOf(Number(res.code)) >= 0) && !res.msg

现在正确处理： return (_ignoreToastCode.indexOf(Number(res.code)) >= 0) && res.msg
```

原本的代码，在请求中传了`ignoreToastCode:[0]`,也是会弹提示,