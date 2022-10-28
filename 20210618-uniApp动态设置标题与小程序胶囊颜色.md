## uniApp动态设置标题与小程序胶囊颜色

封装在Tool.js中，方便全局调用

uniapp根目录的main.js 引入utils/Tool目录下的根文件`index.js`:

```js
//  /utils/Tool/index.js
import * as Tool from './Tool.js'
import * as Auth from './Auth.js'
import * as Equipment from './Equipment.js'
import * as Media from './Media.js'
export default {
	...Media,
	...Equipment,
	...Auth,
	...Tool
}


```

项目根目录的`main.js`引入工具目录：

```js
import Tool from '@/utils/Tool'

// 挂在到Vue原型，全局使用
Vue.prototype.$Tool = Tool
```



```js
/**
 * 修改当前页面的标题栏标题
 */
export const SetPageTitle = (title = '标题') => {
	return new Promise((resolve, reject) => {
		uni.setNavigationBarTitle({
			title,
			success: () => {
				resolve(title)
			},
			fail: erro => {
				reject(erro)
			}
		})
	})
}
/**
 * 修改当前页面的小程序胶囊颜色
 */
export const SetPageBtnColor = (color = '#ffffff') => {
  console.log("color:",color)
	return new Promise((resolve, reject) => {
    if(color=='black'||color=='#000'||color=='#000000') {
      color = '#000000'
    }
    if(color=='white'||color=='#fff'||color=='#ffffff') {
       color = '#ffffff'
    }
		uni.setNavigationBarColor({
			frontColor: color,
      		backgroundColor:'#ffffff',
            animation: {
                duration: 400,
                timingFunc: 'easeIn'
              }
			success: (res) => {
				resolve(res)
			},
          fail: error => {
            reject(error)
          }
		})
	})
}
```

## 动态设置胶囊颜色&自定义导航栏标题颜色

配合NavHeight.js Mixin混入使用

自定义导航栏

```html
 <!-- 自定义导航栏 -->
    <view class="box">
      <li-custom-nav background='transparent' :showBtn="false" title="首页" :showPlace="true" :color="titleColor"></li-custom-nav>
    </view>
```

watch监听isNearTop的值：true/false

```js
watch: {
      isNearTop(n) {
       if (n) {
        uni.setNavigationBarColor({
         frontColor: '#000000',
         backgroundColor: '#ffffff'
        })
        this.titleColor = '#000000'
       } else {
        uni.setNavigationBarColor({
         frontColor: '#ffffff',
         backgroundColor: '#000000'
        })
        this.titleColor = '#ffffff'
       }
      }
    },
```

![image-20210618153937774](https://i.loli.net/2021/06/18/wrSGatWAXcxNuQU.png)

![image-20210618153958775](https://i.loli.net/2021/06/18/OJnEigY8Hzq9pvA.png)

## 封装成公共方法

```js
/**
 * 动态设置胶囊颜色/自定义导航栏标题颜色
 * @isTrue: Boolen 配合NavHeight.js 传入监听isNearTop的值
 * @changeColor: Boolen  是否改变自定义导航栏标题颜色
 * @that: this  传入使用页面的this
 * 
 */
export const changeBtnColor = (isTrue,{changeColor=false,that=this})=> {
  if (isTrue) {
    uni.setNavigationBarColor({
     frontColor: '#000000',
     backgroundColor: '#ffffff'
    })
    if(changeColor) {
      that.titleColor = '#000000'
    }
   } else {
      uni.setNavigationBarColor({
       frontColor: '#ffffff',
       backgroundColor: '#000000'
      })
      if(changeColor) {
        that.titleColor = '#ffffff'
      }
    }
  }
```

