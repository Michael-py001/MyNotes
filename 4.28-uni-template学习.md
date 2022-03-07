# 依赖收集

> 收集依赖后，向外暴露接口，可以在所有页面中引入后使用

## Router收集

```js
// 遍历收集router对象
	// routerItem：每个模块下的router中的index.js暴露的名称
	const routerItem = require(`./${modularName}/router`).default
	for (let pageName in routerItem) {
		_router[`${modularName}.${pageName}`] = {
			...routerItem[pageName],//把原本每个router下的属性添加进去
			// url:"/pages/Common/page/Home/Home"
			// routerItem[pageName]['include'] ->是否含有include（属于哪个模块）
			// 新增一个url属性，代表这个router所在的路径
			url: `/pages/${modularName}/page/${routerItem[pageName]['include'] || pageName}/${pageName}`
		}
	}
```

![image-20210428152305721](C:\Users\PM\AppData\Roaming\Typora\typora-user-images\image-20210428152305721.png)

​											图为依赖搜集后的结果

## API依赖搜集

```js
// 遍历收集api对象
	try {
		// importInfo：所有模块下的api中暴露的api名称
		const importInfo = require(`./${modularName}/api`)
		console.log("importInfo:",importInfo)
		// ModularInfo：每个api下的modularInfo ->controlName: 'user'(声明是哪个模块下的)
		const ModularInfo = importInfo.modularInfo || {}
		// ActionList：importInfo下的default
		const ActionList = importInfo.default || {}
		// 搜集每一项modularName
		_api[modularName] = {
			ModularInfo,
			ActionList
		}
	} catch (e) {}
```

![image-20210428155958609](C:\Users\PM\AppData\Roaming\Typora\typora-user-images\image-20210428155958609.png)

## 请求

utils/Config.js里配置请求的域名：

```js
// 环境：pro生产模式 dev开发模式
	env: 'pro',
	// 所有环境域名
	hostUrl: {
		dev: 'http://dev.com',
		pro: 'https://pro.com',
		mock: 'https://mock.com'
	},
```

`utils/Request/index.js`是一个vue插件，它对外暴露一个install方法，第二个参数是一个可选的选项对象，用于传入插件的配置->options。

### Request插件注册

在`/main.js`根目录下，使用`Vue.use()`方法注册插件：

```js
// 引入请求
Vue.use(Request, {
	...Config,
	hostUrl: Config.hostUrl[Config.env],// 选择Config中的hostUrl，并且区分env环境
	apiObj: api,
	...CodeHandle
})
```

并且在Vue.use()中注册全局请求方法：

```js
// 全局注册请求方法
    Vue.prototype.$Request 
```

每个模块下的api/index.js格式：

```js
export default {
	getValidCode: {
		method: "POST",
		actionName: "getValidCode", //执行名称
	},
	loginByValidCode: {
		method: "POST",
		url: "/index.php?s=/api/user/loginByValidCode"//请求的地址
	},
	loginByAuthPhone: {
		method: "POST",
		url: "/index.php?s=/api/user/loginByAuthPhone"
	},
	updateInfo: {
		api_type: "shop",
		method: "POST",
		url: "/index.php?s=/api/user/updateInfo"
	}
}

```

最终的请求地址会和Config.js的hostUrl合并。

`utils/Request/lib.js:`

> 在Request中提取传入的options信息，

```js
export const Request = (apiName, data = {}, option = {}, _resolve, _reject) => {
	// 提取Option中的信息
	let {
		apiFormat,
		loading,
		loadingText = '加载中',
		header = {},
		method,
		apiType,
		actionName,
		modularName,
		controlName,
		hostUrl, //提取Config设置的根域名
		successCode = [],
		ignoreToastCode = [],
		forceReturn,
		parseData,
		showLoadingHandle,
		closeLoadingHandle,
		catchRequestHandle,
		toastHandle,
		codeHandle
	} = option
	return new Promise((resolve, reject) => {
		// api地址 把hostUrl传进去
		const apiUrl = GetApi(apiName, {
			apiFormat,
			actionName,
			modularName,
			controlName,
			hostUrl
		})
```

`utils/Request/lib.js`中的GetApi方法：

> 

```js
export const GetApi = (apiName, option = {}) => {
	// 域名信息 Config.js中设置的hostUrl
	const hostUrl = ConfigPriority(apiName, option.hostUrl, 'hostUrl')
	// 模块信息
	const modularName = ConfigPriority(apiName, option.modularName, 'modularName')
	// 控制器信息
	const controlName = ConfigPriority(apiName, option.controlName, 'controlName')
	// action信息
	const actionName = ConfigPriority(apiName, option.actionName, 'actionName')
	// api格式
	const apiFormat = ConfigPriority(apiName, option.apiFormat, 'apiFormat')
	// url信息  每个模块下的url信息
	const apiUrl = ConfigPriority(apiName, option.url, 'url')
	// 拼接最后的请求域名地址
	if (apiUrl !== '') {
		return `${hostUrl}${apiUrl}`
	}
	return ApiFormat(apiFormat, {
		hostUrl,
		modularName,
		controlName,
		actionName
	})
}

```

# UI组件引入

1. 在根目录的main.js中引入uview-ui

   ```js
   import uView from "uview-ui"//引入uview-ui
   //使用
   Vue.use(uView);
   ```

   全局注册后直接在页面中使用：

   ```html
   	<view class="list">
   		<view class="stickytop bg">
   			<u-sticky >
   				<u-tabs :list="tabList" :is-scroll="false" :current="active" @change="TabsChange"></u-tabs>
   			</u-sticky>
   		</view>
   		
   		<u-cell-group>
   			<block v-for="(dataitem, dataindex) in list " :key="dataindex">
   				<u-cell-item :title="dataindex + '' " :value="'note' + dataindex" :showArrow="false" />
   			</block>
   		</u-cell-group>
   		<!-- 显示正在加载 去掉还是可以下拉加载，只是没有文字 -->
   		<view class="loadmore">
   			<u-loadmore :status="list_more_status" />
   		</view>
   	</view>
   ```

   

# 引入组件

## 局部注册

1. 引入组件

   ```js
   import ImgUpload from '@/components/form/img-upload';
   import Btn from '@/components/button/index.vue'
   ```

2. 注册组件

   ```js
   components: {
   			ImgUpload,
   			Btn
   		}
   ```

## 全局注册

```js
import uView from "uview-ui"//引入uview-ui 
Vue.use(uView);
```

# 条件编译

```js
// #ifdef APP-PLUS
import * as Push from '@/utils/Push'
// 引入极光推送
Vue.prototype.$Push = Push
// #endif
```

## [条件编译](https://uniapp.dcloud.io/platform?id=条件编译)

条件编译是用特殊的注释作为标记，在编译时根据这些特殊的注释，将注释里面的代码编译到不同平台。

**写法：**以 #ifdef 或 #ifndef 加 **%PLATFORM%** 开头，以 #endif 结尾。

- \#ifdef：if defined 仅在某平台存在
- \#ifndef：if not defined 除了某平台均存在
- **%PLATFORM%**：平台名称

| 条件编译写法                                             | 说明                                                         |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| #ifdef **APP-PLUS** 需条件编译的代码 #endif              | 仅出现在 App 平台下的代码                                    |
| #ifndef **H5** 需条件编译的代码 #endif                   | 除了 H5 平台，其它平台均存在的代码                           |
| #ifdef **H5** \|\| **MP-WEIXIN** 需条件编译的代码 #endif | 在 H5 平台或微信小程序平台存在的代码（这里只有\|\|，不可能出现&&，因为没有交集） |

**支持的文件**

- .vue
- .js
- .css
- pages.json
- 各预编译语言文件，如：.scss、.less、.stylus、.ts、.pug

# 混入Mixins

## 局部混入

```js
import {
		FormValid
	} from '@/utils/Mixin';
export default {
		mixins: [FormValid],//使用mixins
}
```

使用混入后，导入的文件中的所有data,methods等都可以在当前页面调用

## 选项 mixins

`mixins` 选项接收一个混入对象的数组。这些混入对象可以像正常的实例对象一样包含实例选项，这些选项将会被合并到最终的选项中，使用的是和 `Vue.extend()` 一样的选项合并逻辑。也就是说，如果你的混入包含一个 created 钩子，而创建组件本身也有一个，那么两个函数都会被调用。

Mixin 钩子按照传入顺序依次调用，并在调用组件自身的钩子之前被调用。

**示例**：

```js
var mixin = {
  created: function () { console.log(1) }
}
var vm = new Vue({
  created: function () { console.log(2) },
  mixins: [mixin]
})
// => 1
// => 2
```

## 全局API Vue.mixin( mixin )

- **参数**：

  - `{Object} mixin`

- **用法**：

  全局注册一个混入，影响注册之后所有创建的每个 Vue 实例。插件作者可以使用混入，向组件注入自定义的行为。**不推荐在应用代码中使用**。

- **参考**：[全局混入](https://cn.vuejs.org/v2/guide/mixins.html#全局混入)

# easycom

## 1、项目公共组件

> /components

仅在本项目中使用的公共组件。

## 2、跨项目公共组件

> /components/common

跨项目公共组件是指多个项目中**重复使用的组件**，前端开发**不用重复开发这些组件**，只需要从其他项目中把跨项目公共组件复制到当前项目的`/components/common`中。



为了方便前端快速开发，避免重复造轮子，建议公共组件命名需要符合`components/组件名称/组件名称.vue`目录结构，且各个项目配置统一的 easycom 组件匹配规则。

## 3、easycom 组件规范

[easycom 组件规范](https://uniapp.dcloud.io/component/README?id=easycom组件规范)，组件不用引用、注册，直接在页面中使用。

#### 1、编辑 `/pages.json`

以下是自定义的规则。

```json
"easycom": {
  "autoscan": true,
  "custom": {
    "^c-(.*)": "@/components/common/$1/index.vue"
  }
}
```

#### 2、新建公共组件

> /components/common/nav/index.vue

#### 3、在页面中使用公共组件

```html
<template>
    <view>
        <c-nav />
    </view>
</template>
```

![image.png](https://i.loli.net/2021/05/26/i9m3StRHGnwV7gT.png)

![image-20210428180545779](https://i.loli.net/2021/05/26/4LAhMxyfaE8d9ZQ.png)

**自定义easycom配置的示例**

如果需要匹配node_modules内的vue文件，需要使用`packageName/path/to/vue-file-$1.vue`形式的匹配规则，其中`packageName`为安装的包名，`/path/to/vue-file-$1.vue`为vue文件在包内的路径。

```json
"easycom": {
  "autoscan": true,
  "custom": {
    "^uni-(.*)": "@/components/uni-$1.vue", // 匹配components目录内的vue文件
    "^vue-file-(.*)": "packageName/path/to/vue-file-$1.vue" // 匹配node_modules内的vue文件
  }
}
```

**说明**

- `easycom`方式引入的组件无需在页面内`import`，也不需要在`components`内声明，即可在任意页面使用
- `easycom`方式引入组件不是全局引入，而是局部引入。例如在H5端只有加载相应页面才会加载使用的组件
- 在组件名完全一致的情况下，`easycom`引入的优先级低于手动引入（区分连字符形式与驼峰形式）

# 工具类方法使用

## 防抖

公共方式

```js
//首先在data中定义
data () {
	return {
		debounceCheck:()=>{}
	}
}
//在mounted中赋值
mounted () {
    this.debounceCheck = this.$Tool.Deb
}

//methods定义方法
methods: {
    // 检查用户名
    myCheckUser () {
      if (/^([a-zA-Z]|[0-9])(\w|\-)+@[a-zA-Z0-9]+\.([a-zA-Z]{2,4})$/.test(this.willCheckUser)) {
        this.isRegister = 1
      } else {
        console.log('error')
        this.isRegister = 2
      }
    }
  }
```

### 父组件中使用

```html
<text-input v-model="info2" :label="$t('common.login.pw')" :placeholder="$t('common.login.pw_p')" ref="PW" type="password" showForget></text-input>
```



```js
  import { CheckUser } from '@/utils/Mixin' //引入方法

```

```js
data () {
      return {
        currentOption: 1,
        info1: '',
        info2: ''
      }
    },
```

```js
 watch: {
      info1 (val) {
        this.willCheckUser = val
        this.debounceCheck()
      }
    },
```

# 登录流程

```js
    methods: {
      ...mapActions('Common', ['setUserInfo', 'setReviewData']),
      handleLogin (type) {
        if (type === 2) {
          this.$nav('Company.CurrentServe')
        } else {
          this.$nav('Service.ClientIntro')
        }
      },
      async doLogin () {
        let res = await this.$Post('Common.login', {
          username: this.info1,
          password: this.info2,
          type: this.currentOption
        })
        if (res.status) {
          if (res.data.review_status * 1 === 2) {
            await this.setUserInfo(res.data)
            await this.$ShowToast(res.msg)
            this.reset()
            this.handleLogin(res.data.type)
          } else {
            await this.setReviewData({
              review_status: res.data.review_status,
              review_remark: res.data.review_remark,
              username: res.data.username
            })
            this.$nav('Common.Audit')
          }
        } else {
          this.$ShowToast(res.msg)
        }
      },

      reset () {
        this.currentOption = 1
        this.info1 = ''
        this.info2 = ''
      }
    }
```

1. POST请求 提交需要的字段内容

2. 判断返回状态status

3. 使用vuex保存登录信息，

   ```js
   //action
   async setUserInfo ({ commit }, info) {
       await commit('SetUserInfo', info) //通知SetUserInfo执行
     },
   //SetUserInfo保存信息到本地
     SetUserInfo (state, info) {
       SetStorage(Config.tokenStorageKey, info.access_token, 'local')
       SetStorage('userInfo', info, 'local')
       state.userInfo = info
     },
   ```

   **setReviewData** =>

mutation中的SetReviewData 

```js
 SetReviewData (state, data) {
    SetStorage('reviewData', data)
    state.reviewData = data
  },
```

state.js中的reviewData

```js
export default {
  lang: '',
  userInfo: {},
  reviewData: {}
}
```

在getters.js中设置计算属性，可以在computed中使用，动态获取state中的内容

```js
import { GetStorage } from '@/utils/Storage'
export default {
  lang: state => state.lang, //获取state中的lang
  currentUrl: state => state.currentUrl,
  userInfo: (state) => {
    if (Object.keys(state.userInfo).length) {
      return state.userInfo
    } else {
      return GetStorage('userInfo', 'local')
    }
  },
  reviewData: (state) => { ////获取state中的reviewData
    if (Object.keys(state.reviewData).length) {
      return state.reviewData
    } else {
      return GetStorage('reviewData')
    }
  },
  registerServeData: state => state.registerServeData
}

```

## 空间命名

在每个模块下的store/index.js，会搜集state action getter mutations模块，并开启空间命名。

```js
import state from './state'
import getters from './getters'
import actions from './actions'
import mutations from './mutations'

export default {
  namespaced: true,//开启空间命名
  state,
  getters,
  actions,
  mutations
}
```



## getters的使用

一般使用辅助函数，在computed中使用

```js
  import { mapGetters } from 'vuex'

  export default {
    mounted () {
      !Object.keys(this.reviewData).length && this.$nav('Common.Login')
    },
    computed: {
      ...mapGetters('Common', ['reviewData'])
    }
  }
```

## actions的使用

actions.js  

> actions中函数第一个参数包含commit方法，可以唤起mutation中的方法，也是唯一方法.

```js
 async setRegisterData ({ commit }, data) {
    await commit('SetRegisterData', data)
  },
  
  async clearRegisterData ({ commit }) {
    await commit('ClearRegisterData')
  },
  
  async setRegisterServeData ({ commit }, data) {
    await commit('SetRegisterServeData', data)
  },
```

mutations.js 

> mutation中操作的其实也就是state的数据，为了统一，全部使用action来触发，action可以触发异步函数

```js
  export default {
      SetRegisterData (state, data) {
        state.registerData = data
      },
       ClearRegisterData (state) {
        state.registerData = {}
      },
       SetRegisterServeData (state, data) {
        state.registerServeData = data
      },
  }
```



```js
  import { mapActions, mapGetters } from 'vuex'

 export default {
     methods: {
         ...mapActions('Common', ['setRegisterData', 'clearRegisterData', 'setRegisterServeData']),
          async getSupplierInfo () {
            this.currentOption = 2
            let { data } = await this.$Get('Common.getSupplierInfo', {
              username: this.reviewData.username
            })
            this.setRegisterServeData({
              service_area_state: data.service_area_state,
              service_area: data.service_area,
              fixed_project: data.fixed_project,
              office: data.office,
              manager: data.manager
            })
       	  }
         ...
         this.setRegisterData(postData).then(() => { //在actions中使用了async await 这里可以使用.then(...)
            this.$nav(url)
          })
         ...
         
         ...
         reset (){
             this.clearRegisterData()
         }
     }
 }
```

## mutations的使用
