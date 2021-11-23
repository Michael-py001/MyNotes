## 20211117-APP端Video层级问题之subNVues方案实践&Nuve动画

> [uni-app subNVue 原生子窗体开发指南](https://ask.dcloud.net.cn/article/35948)
>
> [subNVues 接口文档](https://uniapp.dcloud.io/api/window/subNVues?id=app-getsubnvuebyid)

APP端某些原生组件，如video，在APP端层级是比html高的，所以无论怎么设置z-index都是没用的，

## 一. pages.json配置

可以在`pages`主包和`subPackages`分包中使用

subNVues:

- id: [String], **全局唯一，不能重复**。
- path: [String], subNVue 子窗体的路径。
- type: [String], 内置的特殊子窗体类型，弹出(popup)和导航(navigationBar)。也可以不设置
- style: [Object], 配置子窗体的位置，背景等样式属性。

**注意事项:**

- `id` 属性是全局唯一的，
- `path` 路径只能是 `nuve` 页面路径
- `type` 属性目前只有导航栏 (`navigationBar`) 和弹出层 (`popup`) 类型，且级别最高，一旦设置 `type` 为 `navigationBar` 或 `popup`， `position` 和 `dock` 的值都会被忽略。
- `position` 为原生子窗体的定位方式。
- `dock` 表示原生子窗体的停靠位置，只有当 `position` 值为 `dock` 时才生效，如 `top`, `bottom`,`right`, `left` 等。
- 在配置中可以使用 upx 单位，方便你进行响应式布局。

在`style`下插入`app-plus`选项，在里面插入`subNVues`:

```json
	{
      "path": "pages/Common/page/Index2/Index2",
      "style": {
        "navigationBarTitleText": "",
        "enablePullDownRefresh": false,
        "navigationStyle": "custom",
        "navigationBarTextStyle": "white",
        "app-plus": {
          "subNVues": [{
            "id": "popup",
            "path": "/components/subNVue/Popup",
            "type": "popup",
            "style": {
              "margin": "auto",
              "width": "80%",
              "height": "600rpx"
            }
          }]
        }
      }
    },
```

如果在分包里设置，需要注意path路径问题，用绝对路径会找不到该文件，需要用相对路径。

```json
{
          "path": "page/AD/AD",
          "style": {
            "navigationBarTitleText": "广告",
            "enablePullDownRefresh": false,
            "app-plus": {
              "subNVues": [{
                "id": "popup",
                "path": "../../components/subNVue/Popup",
                "type": "popup",
                "style": {
                  "margin": "auto",
                  "width": "80%",
                  "height": "600rpx"
                }
              }, {
                "id": "HongBao",
                "path": "../../components/subNVue/HongBao",
                // "type": "popup",
                "style": {
                  "margin": "auto",
                  "width": "100%",
                  "height": "100%",
                  "background": "transparent"
                }
              }]
            }

          }
        },
```

type设置为popup的效果是，会有遮罩层效果，点击任意地方会就会隐藏。为了达到不能点击遮罩层，则不能设置type, 在组件内自己实现遮罩层。style中设置background为透明。

## subNVue 子窗体书写

`subNVue` 子窗体引用的是 `nvue` 页面。所以只需要书写 `nvue` 页面。

需要注意的是，`nvue` 与 `vue` 页面的开发注意事项。两者开发起来还是有一些区别。

[weex文档](http://doc.weex.io/zh/docs/components/text.html#%E5%B1%9E%E6%80%A7)

[weex 通用样式以及需要注意的问题](http://www.ptbird.cn/weex-common-style-and-problems.html)

**相关参考**

- 使用 `nvue` 开发注意事项：https://uniapp.dcloud.io/nvue-outline
- 使用 `vue` 开发注意事项：https://uniapp.dcloud.io/vue-basics

使用nvue也就是weex那一套，坑还是很多的，列一下我遇到的：

- CSS只能使用单选择器，比如`.a` 不能`.a .b`，不能使用less,scss，也就不能嵌套写了
- 文本内容需要用专用的`text`标签，不然无法使用font-size等设置字体的属性
- CSS属性不能继承到子节点，需要每个class写一遍
- 某些标签不能使用选择器,比如`.body`
- 不支持CSS的animate动画，只能用简单的transition
- 设置的class不能用平常vue中`:class={xx:xx}`的方式动态绑定，需要设置两个class`:class={a:xx} :class="{b:xx}"`，不需要 某个效果时，在另外的class中设置属性
- 实现动画推荐使用JS，有原生的`animation`和第三方库`Binding`(下面会介绍)

## 怎么在页面中使用 subNVue 子窗体

在 `pages.json` 中增加完配置，也写好了 `subNVue` 子窗体，接下来就是在 `vue`/`nvue` 页面中使用了。 在 `vue` 和 `nvue` 页面中使用方式是一样的，这里以 `vue` 页面为例进行说明:

**在页面中打开和关闭 subNVue 子窗体**

这个id就是在pages.json中设置的id，而且不需要在标签上设置id，这个是两回事，网上的文章有的说需要在标签设置同名的id，纯属扯淡。

```javascript
// 通过 id 获取 nvue 子窗体  
const subNVue = uni.getSubNVueById('map_widget')  
// 打开 nvue 子窗体  
subNVue.show('slide-in-left', 300, function(){  
    // 打开后进行一些操作...  
    //   
});  
// 关闭 nvue 子窗体  
subNVue.hide('fade-out', 300)
```

**动态修改 subNVue 子窗体位置，大小**

```javascript
subNVue.setStyle({  
    top: '100px',  
    left: '20px',  
    width: '100px',  
    height = '50px',  
})
```

### 推荐使用页面通讯完成与子窗体通讯(新增)

关于页面通讯的内容详见: [页面通讯指南](https://ask.dcloud.net.cn/article/36010)

**通讯实现方式**

还是用$on和$emit的方式，因为无法传props

```javascript
// 在 subNVue/vue 页面注册事件监听方法  
// $on(eventName, callback)  
uni.$on('page-popup', (data) => {  
    vm.title = data.title;  
    vm.content = data.content;  
})  

// 在 subNVue/vue 页面触发事件  
// $emit(eventName, data)  
uni.$emit('page-popup', {  
    title: '我是一个title',  
    content: '我是data content'  
});
```

使用页面通讯时注意事项： **要在页面卸载前，使用 uni.$off 移除事件监听器。**

```js
beforeDestroy(){
	uni.$off('page-popup')
}
```

## nvue实现动画

因为用的是weex，所以动画大多数需要用JS实现，因为不支持css animation 

### 引入内置原生插件

> https://uniapp.dcloud.io/api/extend/native-plugin?id=requirenativeplugin



### 使用原生的animation

案例一：循环缩放动画

![animation1](https://i.loli.net/2021/11/19/LloeUihT8E5GMuq.gif)

使用JS控制，我是使用`setInterval`，用两个class反复设置

```js
this.interval = setInterval(() => {
	this.bigAnimate = !this.bigAnimate
}, 500)
```

```html
<image class="img-front-btn " @click="open"
        :class="{'bigAnimate':bigAnimate,'noBigAnimate':!bigAnimate}"
        src="http://hongren-online.oss-cn-guangzhou.aliyuncs.com/20211111185221aa71a1236.png" mode="aspectFill"></image>
```

```css
 .bigAnimate {
    transform: scale(1.1);
  }

  .noBigAnimate {
    transform: scale(0.9);
  }
```

如果是普通的vue，直接设置animation动画就可以实现了

```css
 @keyframes big {
    0% {
      transform: scale(0.9);
    }

    100% {
      transform: scale(1.1);
    }
  }
```

### 案例二

首先把正面/反面内容定位好，反面的css设置`opicity:0 rotateY(180deg) `，等会转过来之后设置` rotateY(0deg)`，而且要用`v-if`判断转过来后隐藏正面，如果不隐藏正面，转过来之后还是正面在最顶层，这是因为安卓中层级是根据渲染顺序排的，不存在`z-index`，用`opicity`也不生效，所以要显示背面，必须要隐藏正面。

> [weex animation动画](https://www.bookstack.cn/read/weex/references-modules-animation.md)
>
> [动画在线调试](http://dotwe.org/vue/54acacaf2c9cbd72fca974b3bc2c474c)

![animation2](https://i.loli.net/2021/11/19/RTlACPorVQaFzp6.gif)

```js
const animation = uni.requireNativePlugin('animation')
open() {
        this.bigAnimate = false
        let that = this
        setTimeout(() => {
          that.runAnimateOne = true
        }, 250)
        animation.transition(this.$refs.front, {
          styles: {
            transform: `rotateY(180deg)`,
          },
          duration: 500, //ms
          timingFunction: 'ease',
          needLayout: false,
          delay: 0 //ms
        }, () => {

        })
        animation.transition(this.$refs.back, {
          styles: {
            transform: `rotateY(0deg)`,
          },
          duration: 500, //ms
          timingFunction: 'ease',
          needLayout: false,
          delay: 0 //ms
        }, function() {})
      
```

### 案例3

动画过程：先上移，后下移，最后隐藏，两段动画，要设置两个部分，使用定时器实现

![animation3](https://i.loli.net/2021/11/19/xDPfiocsmRBVCX7.gif)

```js
const animation = uni.requireNativePlugin('animation')

clickGetMoney() {
    animation.transition(this.$refs.content, {
        styles: {
            transform: `translateY(-50rpx)`,
        },
        duration: 200, //ms
        timingFunction: 'ease',
        needLayout: false,
        delay: 0 //ms
    }, function() {})
    setTimeout(() => {
        let vm = this
        animation.transition(this.$refs.content, {
            styles: {
                transform: `translateY(2000rpx)`,
            },
            duration: 600, //ms
            timingFunction: 'ease',
            needLayout: false,
            delay: 0 //ms
        }, function() {
            console.log("vm", vm.$emit)
            uni.$emit('hideHongBao')
        })
    }, 200)
},
```

### 使用bindingx

> [参考案例](https://blog.csdn.net/qq_33718648/article/details/93393596)

可以实现交互动画，高性能，不卡顿，适合复杂动画

```js
   bind(e) {
       const Binding = weex.requireModule('bindingx');

       Binding.bind({
           eventType: 'scroll',
           anchor: this.getEl(this.$refs.list1),
           props: [{
               element: this.getEl(this.$refs.block), //动画元素  
               property: 'transform.rotateY', //动画属性  
               expression: '200-y' //表达式 说明了y从0-400,对应的值是1-0    
           },
                   {
                       element: this.getEl(this.$refs.block), //动画元素  
                       property: 'translateY', //动画属性  
                       expression: 'y<200?1+y:2-(y-200)/200' //说明了y从0-200,对应的值是1-2  
                   }
                  ]
       })
   },
```

