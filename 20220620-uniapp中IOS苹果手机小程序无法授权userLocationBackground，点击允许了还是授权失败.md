# 20220620-uniapp中IOS苹果手机小程序无法授权userLocationBackground，点击允许了还是授权失败

在某些场景，比如骑手配送，如果需要实时更新位置信息，在地图上显示骑手头像，这就需要频繁获取手机的定位，微信小程序对此是有限制的，不能写一个轮询，频繁调用`wx.getLocation`，只能使用位置更新接口，比如：

- wx.startLocationUpdate  开启小程序进入前台时接收位置消息
- wx.startLocationUpdateBackground  开启小程序进入前后台时均接收位置消息

像上面的场景，需要后台也更新位置信息，这时候需要引导用户授权位置信息。

> [wx.startLocationUpdateBackground(Object object) | 微信开放文档 (qq.com)](https://developers.weixin.qq.com/miniprogram/dev/api/location/wx.startLocationUpdateBackground.html)

![image-20220620184606738](https://s2.loli.net/2022/06/20/9fpxmMFRyYiP1kX.png)

官方文档中也有提醒：

- 需在 app.json 中配置`requiredBackgroundModes: ['location']`后使用

然而安卓手机和苹果不同，安卓手机不需要设置这个也能正常授权，但是在苹果手机必须要设置，而且在uniapp中，需要在`manifest.json`中的源码视图中找到`mp-weixin`中加入上面的代码，这样才能在苹果手机上授权。

然而苹果手机还会出现，明明点击允许离开后获取定位了，但事实上并没有设置成功，这是因为只是设置允许了，实际的设置需要用户手动设置。

这时候需要调用`wx.openSetting`，让用户自己去设置，根据设置回调信息进行再次判断和提示。