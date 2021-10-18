## 20210916-uni-templdate逸店请求模式使用

## API格式

逸店模式需要加`api_type: "shop"`，Request会做特别的处理

```json
BargainGoods: {
		api_type: "shop",
		method: "GET",
		url: '/api/apps.bargain/activeDetail'
	},
```

## 特殊处理

```js

import {
	getShopRequestData
} from './shop.js'

/**
 *  商城请求参数处理
 */
export const handleApi = (reqData, apiType = 'ztbcms', _method = "get") => {
	if (apiType == 'shop') {
		if (_method == 'get') {
			reqData['data'] = {
				...reqData['data'],
				...getShopRequestData()
			}
		} else {
			console.log("_method", _method)
			let url = reqData['url']
			let shopRequestData = getShopRequestData()
			let shopRequestDataString = []
			for (let key in shopRequestData) {
				shopRequestDataString.push(key + "=" + shopRequestData[key])
			}
			if (url.indexOf('?')) {
				reqData['url'] = url + "&" + shopRequestDataString.join("&")
			} else {
				reqData['url'] = url + "?" + shopRequestDataString.join("&")
			}
		}

	}
	return reqData
}
```

### getShopRequestData

```js
import Config from '../Config.js'

/**
 * 获取shop 请求统一参数
 */
export const getShopRequestData = () => {
	return {
		wxapp_id: getShopWxappId(),
		token: getShopToken(),
		version: getShopVersion(),
		env_version: getEnvVersion(),
		platform: getPlatform()
	}
}

/**
 *  获取店铺的wxapp_id
 */
export const getShopWxappId = () => {
	if (typeof Config.shop != "undefined") {
		return Config.shop.wxapp_id
	} else {
		return 10001
	}
}
```

### Config.js

```js
	shop: {
		// 新加坡
		appid: "wx9dbd8d0dd1d0038f",
		wxapp_id: "10019",
		aldKey: "",
		version: "4.1.1", // 以这个版本为准
		// 小程序 wechat 网页 h5
		// #ifdef MP-WEIXIN
		platform: "wechat",
		// #endif
		// #ifdef H5
		platform: "mini_h5"
		// #endif
	}
}
```

## 兼容门店链接跳转

```js
@/pages/index.js
export const navFromShop = (url) => {
	if (!url) {
		return
	}

	let splitMinArray = url.split(":")
	if (splitMinArray[0] && splitMinArray[0] == 'min') {
		// #ifdef MP-WEIXIN
		let appId = splitMinArray[1]
		let path = splitMinArray[2]
		uni.navigateToMiniProgram({
			appId,
			path
		})
		// #endif
		return
	}

	let splitArray = url.split("?")
	let path = splitArray[0]
	let query = ""
	if (splitArray[1]) {
		query = getQuery(splitArray[1])
	}
	if (ShopUrltoRoute[path] && ShopUrltoRoute[path] !== 'undefined') {
		let route = ShopUrltoRoute[path]
		nav(route, query)
	} else {
		//默认在前面加上/绝对路径
		if (url.indexOf('/') !== 0) {
			url = "/" + url
		}
		uni.navigateTo({
			url
		})
	}
}
```

引入

```js
import {
	nav,
	api,
	navFromShop
} from '@/pages/Index'
//兼容门店链接跳转
Vue.prototype.$navFromShop = navFromShop
```

### 跳转链接 ShopUrltoRoute.js

```js
//@utils/ShopUrltoRoute.js
export default {
	"pages/article/index": "Article.Index",
	"pages/address/index": "User.Address",
	"pages/coupon/coupon": "User.Coupon_v2",
	"pages/user/coupon/coupon": "User.MyCoupon",
	"pages/dealer/index/index": "Dealer.Index",
	"pages/bargain/index/index": "Bargain.Index",
	"pages/sharing/order/index": "Sharing.OrderList",
	"pages/user/help/index": "Store.HelpCenter",
	"pages/custom/index": "Common.Custom",
	"pages/live/index/index": "Store.Live",
	"pages/category/list": "Goods.Search_v2",//搜索页
	"pages/category/index": "Common.Category_v2",//分类
	// "pages/category/list": "Goods.CategoryList",
	"pages/goods/index": "Goods.Detail",
	"pages/search/index": "Goods.Category_v2",//搜索页
	"pages/flow/index": "Common.Cart",
	"pages/user/index": "User.Index",
	"pages/order/index": "User.Order",
	"pages/sharing/goods/index": "Sharing.Goods",
	"pages/sharing/active/index": "Sharing.Active",
	"pages/article/detail/index": "Article.Detail",
	"pages/shop/detail/index": "Store.ShopList",
	"pages/bargain/goods/index": "Bargain.Goods",
	"pages/bargain/task/index": "Bargain.Task",
	"pages/sharp/index/index": "Sharp.Index",
	"pages/sharp/goods/index": "Sharp.Goods_v2",
	"pages/presell/order/index": "Presell.OrderList",
	"pages/user/resident": "User.Resident",
	"pages/store/detail": "Store.ShopDetail_v2"
}

```

