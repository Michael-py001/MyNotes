# 20211228-微信H5 JSDK调用chooseImages接口问题

## 问题

H5页面选择手机本地图片，如果用`uni.chooseImage`，选择拍照或者选择了较大的图片时，部分手机会卡顿甚至崩溃。原因可能是浏览器不允许缓存过大的图片。

## 解决方案

调用微信API，并加上压缩参数



封装的方法

获取JSDK配置，授权

```js
var wx = require('jweixin-module')
const initWeixin = ()=> {
  console.log("initWeixin初始化")
  if (!isWechat()) {
  	ShowToast("请在微信浏览器中打开")
  	return
  }
  console.log("url--:",Vue.prototype.$GetStorageSync('scanUrl'))
  let url = /(Android)/i.test(navigator.userAgent) ? location.href.split('#')[0] :GetStorageSync('scanUrl')
  // 兼容苹果手机要点两次才跳转
  uni.getSystemInfo({
    success(res) {
      if(res.errMsg ==='getSystemInfo:ok' && res.platform.toLowerCase() === 'ios') {
        console.log("苹果手机")
        url = url.split("#")[0]
      }
    }
  })
  console.log("url:", url)
  ShowLoading('请求中……')
  return Request('User.GetWxSDK',{
    url: url,
  }).then(res=>{
    console.log("res--:",res)
    new Promise((resolve, reject) => {
      // JS接口列表
      let jsApiList = [
        'chooseWXPay',
        'openLocation',
        'getLocation',
        'checkJsApi',
        'scanQRCode',
        'chooseImage',
        'updateAppMessageShareData',
        'updateTimelineShareData'
      ]
      if(res.status && res.data) {
        let config = res.data.config
        wx.config({
          debug: config.debug||false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
          appId: config.appId, // 必填，公众号的唯一标识
          timestamp: config.timestamp, // 必填，生成签名的时间戳
          nonceStr: config.nonceStr, // 必填，生成签名的随机串
          signature: config.signature,// 必填，签名
          jsApiList: jsApiList // 必填，需要使用的JS接口列表
        });
        wx.ready(()=>{
          console.log("wx.ready---")
          resolve(true)
        })
        wx.error((err)=>{
          console.log("wx.error:",err)
          reject(err)
        })
      }
      else {
        reject(res)
      }
     CloseLoading()
    }).catch((err)=>{
      CloseLoading()
    })
    // this.$SetStorage('o')
  }).catch(err=>{
    console.log("err--:",err)
    CloseLoading()
  })
}
```

调用选择相册API

```js
const ChooseImage = ({
	count = 9,
	sizeType = ['original', 'compressed'],
	sourceType = ['album', 'camera']
} = {})=>{
  if (!isWechat()) {
  	ShowToast("请在微信浏览器中打开")
  	return
  }
  return new Promise((resolve,reject)=>{
    let needResult = needResult
    let scanType = scanType
    initWeixin().then((res)=>{
      console.log("initWeixin----:",res)
      let result = res
      wx.chooseImage({
        	sizeType:sizeType,
        sourceType:sourceType,
        success: (res) => {
        	console.log("scanQRCode", "success", res)
        	resolve(res)
        },
        fail: (err) => {
        	console.log("scanQRCode", "fail", err)
        	reject(err)
        }
      })
    })
  }).catch(err=>{
      ShowToast("调用接口失败",err)
    })
}
```

根据本地图片id 获取base64图片数据

```js
// 根据本地图片id 获取base64图片数据
const GetLocalImgData = (id)=>{
  return new Promise((resolve,reject)=>{
    wx.getLocalImgData({
      localId: id, // 图片的localID
      success: function (res) {
        var localData = res.localData; // localData是图片的base64数据，可以用img标签显示
        if(!localData.includes('data:image/png;base64,')) {
          localData = 'data:image/png;base64,'+localData
        }
        resolve(localData)
      }
    })
  })
}
```

## 组件内使用

```js
uploadImg() {
        this.$wxjs.ChooseImage({sizeType:['compressed']}).then(async res=>{
          let urlList = res.localIds //得到的是id数组
          let base64List = []
          for(let i=0;i<urlList.length;i++) {
            await this.$wxjs.GetLocalImgData(urlList[i]).then(res=>{ //根据id获取图片数据
              base64List.push(res)
            })
          }
          console.log("base64List:",base64List)
          this.$emit('uploadImg',base64List)
        }).catch(err=>{
           console.log("取消：",err)
        }) 
      },
```

最终效果是在手机上没有卡崩溃了，能够正常使用了
