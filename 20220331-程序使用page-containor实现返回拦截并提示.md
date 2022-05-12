# 20220331-小程序使用page-containor实现返回拦截并提示.md

> 官方文档：[page-container | 微信开放文档 (qq.com)](https://developers.weixin.qq.com/miniprogram/dev/component/page-container.html)

## 前置需求

用户修改了数据但是没有点击保存的情况下，通过手机的返回或者点击小程序左上角返回，需要弹出一个提示是否保存。

小程序和APP不同，不能使用系统的API进行拦截，所以只能依靠微信小程序提供的能力。

翻看了文档，发现只有一个组件是可用于拦截返回的，就是`page-container`。

官方文档的描述：

> 小程序如果在页面内进行复杂的界面设计（如在页面内弹出半屏的弹窗、在页面内加载一个全屏的子页面等），用户进行返回操作会直接离开当前页面，不符合用户预期，预期应为关闭当前弹出的组件。 为此提供“假页”容器组件，效果类似于 `popup` 弹出层，页面内存在该容器时，当用户进行返回操作，关闭该容器不关闭页面。返回操作包括三种情形，右滑手势、安卓物理返回键和调用 `navigateBack` 接口。

简单来说就是相当于显示一个弹出层，类似于组件库中的Popup组件。但是在返回时，不会把整个页面退出。

**主要API如下**

| 属性             | 类型        | 默认值 | 必填 | 说明             | 最低版本                                                     |
| :--------------- | :---------- | :----- | :--- | :--------------- | :----------------------------------------------------------- |
| show             | boolean     | false  | 否   | 是否显示容器组件 | [2.16.0](https://developers.weixin.qq.com/miniprogram/dev/framework/compatibility.html) |
| bind:beforeenter | eventhandle |        | 否   | 进入前触发       | [2.16.0](https://developers.weixin.qq.com/miniprogram/dev/framework/compatibility.html) |
| bind:beforeleave | eventhandle |        | 否   | 离开前触发       | [2.16.0](https://developers.weixin.qq.com/miniprogram/dev/framework/compatibility.html) |

实际测试中，show是控制显示，一但检测到返回操作，弹窗就会隐藏，无需手动设置show为false。



![page](https://s2.loli.net/2022/04/23/rdifC9bvTehxQRn.gif)

## 分析

了解清楚这个组件后，再来看下我们的需求，怎么把这个组件的功能用在这个需求上。

这个组件的机制是：弹出显示后-->返回-->组件隐藏，

我们的需求：页面显示-->返回-->弹出提示

根据这两者分析，我们可以在一开始把页面写在`page-container`里面，把提示放在外面。这样按返回时，页面就会消失，弹窗就会显示。

## 代码实现

```html
<template>
<div>
  <page-container :duration="0" :show="showExit" @beforeleave="open">
    <scroll-view :scroll-y="true" class="scroll" @refresherrefresh="refresh"  :refresher-triggered="pullDownStatus">
      <TabBars  :me="true" :list="tab_list" :tab_active.sync="tab_index" tab_height="105"/>
      <div class="DeviceParams">
      
        <div v-show="tab_index==0">
          <SystemParams :id="id" ref="SystemParams"></SystemParams>
        </div>
        
        <div v-show="tab_index==1">
          <UserParams :isNotSave.sync="isNotSave" :id="id" ref="UserParams"></UserParams>
        </div>
        
      </div>
      <div class="refresh" @click="refresh">
        <img :class="{'animate':pullDownStatus}" src="/static/images/refresh.png" alt="">
      </div>
    </scroll-view>
  </page-container>
  <ConfirmPopup :showCancel="true" confirmText="确认"  ref="exitPopup" @confirm="confirmPopup" @cancel="$nav('back')">
      <div class="popup-content">
        您有修改未保存，是否保存？
      </div>
  </ConfirmPopup>
  <Progress :finish.sync="finish" ref="Progress"></Progress>
</div>
  
</template>
```

实际中发现，`page-container`里面不能像正常写页面那样可以滑动，所以这里用`scroll-view`做成一个滚动区域，这样就跟正常页面没有什么大的区别了，目前发现的不足就是，不能用自带的页面下拉刷新，但是可以使用`scroll-view`的自定义下拉实现。
